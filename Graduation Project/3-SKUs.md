تمام، خليني أشرحلك كل حاجة عن الـ `Sku` من الأساس – إيه هي، ليه موجودة، وإزاي هتبنيها كـ Resource كامل في NestJS.
sku is an identifier for the product 
SKU = **Stock Keeping Unit** — يعني الكود الفريد لكل منتج/صنف في المخزون. مش اسم المنتج، دا identifier زي رقم قومي للمنتج.

مثال: عندك "تيشيرت أزرق مقاس L" — دا SKU لوحده، و"تيشيرت أزرق مقاس M" SKU تاني، حتى لو المنتج الأساسي واحد. في مشروعك المشروع مبسط شوية (مش هيبقى فيه variants معقدة)، فكل صف في جدول `skus` = صنف عنده:

- اسم، كاتيجوري، تكلفة (cost)، سعر بيع (price)
- reorder threshold (لما الكمية توصل تحته، النظام يعتبره "محتاج إعادة طلب")
- الكمية الحالية (لكن دي مش مصدر الحقيقة الوحيد — انت هتتابعها عن طريق **ledger** من الحركات، مش overwrite مباشر)
- ---
- 
## إيه هي الـ Sku أصلاً؟

**SKU** = Stock Keeping Unit، يعني ببساطة **"المنتج"** في نظامك. كل صف في جدول الـ `skus` بيمثل منتج واحد بتبيعه أو بتخزنه (زي "لابتوب Dell XPS 13" أو "قلم جاف أزرق").

هي **قلب النظام كله** — كل حاجة تانية في المشروع بترجع ليها بشكل أو بآخر:

- حركة المخزون (`StockMovement`) بترجع لـ SKU معينة
- المورد (`Vendor`) بيبيع كذا SKU في الكاتالوج بتاعه
- أمر الشراء (`PurchaseOrder`) بيحتوي على lineItems كل واحد فيهم بيرجع لـ SKU
- الـ Forecasting Agent بيتوقع الطلب **لكل SKU لوحدها**
- الـ Reorder Agent بيراقب مستوى مخزون **كل SKU** مقابل حد إعادة الطلب بتاعها

## الحقول المطلوبة (من مستند الـ features والـ schema)

|الحقل|النوع|الشرح|
|---|---|---|
|`sku`|string, unique|الكود التعريفي (زي "SKU-118")|
|`name`|string|اسم المنتج|
|`category`|string|الفئة (مفيدة للتقارير و"أرخص مورد للفئة دي")|
|`cost`|decimal|سعر الشراء (التكلفة)|
|`price`|decimal|سعر البيع|
|`reorderThreshold`|number|الحد اللي لو المخزون نزل تحته، يتفعّل تنبيه/Reorder Agent|
|`currentQuantity`|number|الكمية الحالية — **دي denormalized/cache**، المصدر الحقيقي للحقيقة هو ledger الـ `StockMovement`|
|`preferredVendor`|علاقة بـ Vendor|المورد المفضّل لإعادة الطلب|

**نقطة مهمة جدًا من المستندات:** `currentQuantity` **مينفعش تتحدث مباشرة أبدًا**. كل تغيير في الكمية لازم يحصل عن طريق إدخال صف جديد في `stock_movements` (ledger)، والـ `currentQuantity` بتتحدث كـ نتيجة لعملية الإدخال دي (transaction واحدة). ده أساس الـ "Auditability" اللي المستندات بتركز عليه.

## إزاي هتبنيها كـ Resource كامل — الخطوات بالترتيب

### 1. الـ Entity

```ts
// src/inventory/entities/sku.entity.ts
import { Entity, Column, ManyToOne, JoinColumn } from 'typeorm';
import { AbstractEntity } from '../../shared/base.entity';
import { Vendor } from '../../vendors/entities/vendor.entity';

@Entity('skus')
export class Sku extends AbstractEntity {
  @Column({ unique: true })
  sku!: string;

  @Column()
  name!: string;

  @Column({ nullable: true })
  category?: string;

  @Column('decimal', { precision: 10, scale: 2 })
  cost!: number;

  @Column('decimal', { precision: 10, scale: 2 })
  price!: number;

  @Column({ default: 0 })
  reorderThreshold!: number;

  @Column({ default: 0 })
  currentQuantity!: number;

  @ManyToOne(() => Vendor, { nullable: true })
  @JoinColumn({ name: 'preferred_vendor_id' })
  preferredVendor?: Vendor;
}
```

### 2. الـ DTOs (شكل البيانات اللي بتدخل/تطلع من الـ API)

```ts
// src/inventory/dto/create-sku.dto.ts
import { IsString, IsNumber, IsOptional, Min } from 'class-validator';

export class CreateSkuDto {
  @IsString()
  sku: string;

  @IsString()
  name: string;

  @IsOptional()
  @IsString()
  category?: string;

  @IsNumber()
  @Min(0)
  cost: number;

  @IsNumber()
  @Min(0)
  price: number;

  @IsNumber()
  @Min(0)
  reorderThreshold: number;

  @IsOptional()
  @IsString()
  preferredVendorId?: string;
}
```

```ts
// src/inventory/dto/update-sku.dto.ts
import { PartialType } from '@nestjs/mapped-types';
import { CreateSkuDto } from './create-sku.dto';

export class UpdateSkuDto extends PartialType(CreateSkuDto) {}
```

**ليه DTO منفصل عن الـ Entity؟** عشان متسمحش لحد يبعتلك `currentQuantity` مباشرة من الـ API — الكمية بتتغير بس عن طريق `stock movements`، مش عن طريق تعديل مباشر على الـ SKU.

### 3. الـ Service (المنطق)

```ts
// src/inventory/services/sku.service.ts
@Injectable()
export class SkuService {
  constructor(
    @InjectRepository(Sku) private skuRepo: Repository<Sku>,
  ) {}

  create(dto: CreateSkuDto) {
    const sku = this.skuRepo.create(dto);
    return this.skuRepo.save(sku);
  }

  findAll() {
    return this.skuRepo.find({ relations: ['preferredVendor'] });
  }

  findOne(id: string) {
    return this.skuRepo.findOneOrFail({ where: { id }, relations: ['preferredVendor'] });
  }

  findLowStock() {
    return this.skuRepo
      .createQueryBuilder('sku')
      .where('sku.currentQuantity <= sku.reorderThreshold')
      .getMany();
  }

  update(id: string, dto: UpdateSkuDto) {
    return this.skuRepo.update(id, dto);
  }
}
```

لاحظ إن `findLowStock()` ده اللي هيغذي الداشبورد و Reorder Agent بعدين.

### 4. الـ Controller (الـ API endpoints)

```ts
// src/inventory/controllers/sku.controller.ts
@Controller('skus')
export class SkuController {
  constructor(private skuService: SkuService) {}

  @Post()
  create(@Body() dto: CreateSkuDto) {
    return this.skuService.create(dto);
  }

  @Get()
  findAll() {
    return this.skuService.findAll();
  }

  @Get('low-stock')
  findLowStock() {
    return this.skuService.findLowStock();
  }

  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.skuService.findOne(id);
  }

  @Patch(':id')
  update(@Param('id') id: string, @Body() dto: UpdateSkuDto) {
    return this.skuService.update(id, dto);
  }
}
```

### 5. الـ Module (يربط كل حاجة ببعض)

```ts
// src/inventory/inventory.module.ts
@Module({
  imports: [TypeOrmModule.forFeature([Sku])],
  controllers: [SkuController],
  providers: [SkuService],
  exports: [SkuService], // مهم عشان Agents Module هيحتاجه بعدين
})
export class InventoryModule {}
```

لازم تضيفه في `app.module.ts` جوه الـ `imports`.

## ترتيب البناء المقترح لنفسك

1. الـ Entity الأول (وده لازم تتأكد إن الـ `Vendor` entity موجودة الأول لو هتربطها بيها، وإلا اعمل العلاقة بعدين)
2. الـ DTOs
3. الـ Service
4. الـ Controller
5. الـ Module + تسجيله في `app.module.ts`
6. تجرب على Postman/Thunder Client: POST سكو جديدة، GET كل السكوات، GET low-stock

بعدين لما تخلص الـ `Sku`، هتعمل نفس النمط بالظبط لـ `Vendor` و `StockMovement` — كلهم هيتبعوا نفس الشكل (`Entity → DTO → Service → Controller → Module`).

عايز أساعدك دلوقتي في `StockMovement` (اللي هي الـ ledger اللي بتحدّث الـ `currentQuantity`)، ولا لسه عندك أسئلة على الـ `Sku` نفسها؟