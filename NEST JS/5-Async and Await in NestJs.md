

## أول حاجة — فهم المشكلة

**Node.js single-threaded** — يعني فيه thread واحدة بس بتخدم كل الـ requests.

تخيل إن عندك كافيه وفيه **نادل واحد بس.** جه زبون وطلب قهوة — النادل راح المطبخ واستنى القهوة تتعمل. في الوقت ده، جه زبون تاني — مفيش حد يخدمه. بيستنى. جه زبون تالت — بيستنى.

ده بالظبط اللي بيحصل في Node.js لو شغلت code بـ sync — الـ thread بتوقف وكل الـ requests بتستنى.

---

## الحل — Async / Await

النادل الذكي مش بيوقف في المطبخ. بيطلب القهوة، يرجع يخدم ناس تانية، ولما القهوة تجهز يرجع يجيبها.

ده بالظبط معنى **async/await** — الـ thread مش بتوقف. بتروح تخدم request تاني وبترجع لما الرد يجي.

---

## القاعدة اللي هتفضل معاك

```
بتتكلم مع حاجة برا الكود؟  →  async + await
DB / API / File / Cache

بتعمل حاجة جوا الكود بس?  →  عادي بدون async
حسابات / string / array
```

---

## المثال العملي الكامل

```typescript
// ━━━ Entity ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

@Entity()
export class User {

  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column()
  email: string;
}


// ━━━ Service ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

@Injectable()
export class UserService {

  constructor(
    @InjectRepository(User)
    private repo: Repository<User>,
  ) {}


  // ✅ بتكلم DB  →  async
  async findAll(): Promise<User[]> {
    return await this.repo.find();
  }


  // ✅ بتكلم DB + بترمي error لو مش موجود  →  async
  async findOne(id: number): Promise<User> {
    const user = await this.repo.findOneBy({ id });

    if (!user) throw new NotFoundException(`مفيش يوزر بالـ id ده: ${id}`);

    return user;
  }


  // ✅ بتحفظ في DB  →  async
  async create(dto: CreateUserDto): Promise<User> {
    const user = this.repo.create(dto);
    return await this.repo.save(user);
  }


  // ❌ مش بتكلم DB ولا أي حاجة برا  →  مش محتاج async
  formatWelcome(user: User): string {
    return `Welcome, ${user.name}!`;
  }
}


// ━━━ Controller ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

@Controller('users')
export class UserController {

  constructor(private userService: UserService) {}


  // NestJS بيمسك الـ exceptions تلقائي  →  مش محتاج try/catch هنا
  @Get()
  async findAll(): Promise<User[]> {
    return await this.userService.findAll();
  }


  @Get('/:id')
  async findOne(@Param('id') id: number): Promise<User> {
    return await this.userService.findOne(id);
  }


  @Post()
  async create(@Body() dto: CreateUserDto): Promise<User> {
    return await this.userService.create(dto);
  }
}
```

---

## Promise.all — لما عندك عمليتين مستقلتين

```typescript
// ❌ غلط — بتستنى الأولى تخلص عشان تبدأ التانية
// وقت التنفيذ: 300ms + 200ms = 500ms
async getProfile(userId: number) {
  const user   = await this.userService.findOne(userId);     // استنى 300ms
  const orders = await this.orderService.findByUser(userId); // استنى 200ms
  return { user, orders };
}


// ✅ صح — الاتنين بيشتغلوا مع بعض في نفس الوقت
// وقت التنفيذ: max(300ms, 200ms) = 300ms بس
async getProfile(userId: number) {
  const [user, orders] = await Promise.all([
    this.userService.findOne(userId),
    this.orderService.findByUser(userId),
  ]);
  return { user, orders };
}
```

> **⚠️ متستخدمش Promise.all** لو العملية التانية محتاجة نتيجة الأولى. في الحالة دي لازم تنفذهم ورا بعض.

---

## الخلاصة

```
Node.js  →  thread واحدة بس

المشكلة  →  لو وقفت تستنى DB، كل الـ requests بتستنى

الحل     →  async/await
            الـ thread مش بتوقف
            بتروح تخدم request تاني وبترجع لما الرد يجي

القاعدة  →  DB / API / File / Cache  =  async + await
            حسابات / string / array  =  عادي بدون async

Bonus    →  عمليتين مستقلتين؟  →  Promise.all
```