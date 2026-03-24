

- Observable
    
- subscribe (next / error / complete)
    
- Service + Model + Reactive Form
    
- CRUD كامل بطريقة clean
    

---

# 🔥 أولاً: يعني إيه Observable في Angular؟

Angular بيستخدم **RxJS**

### 💡 الفكرة ببساطة:

`Observable` = مصدر بيانات بيبعتلك values مع الوقت

مش زي Promise ❌ (مرة واحدة وخلاص)  
لكن ممكن:

- يبعت أكتر من value
    
- أو error
    
- أو complete
    

---

# 🧠 تخيلها كده:

```ts
this.http.get()
```

دي مش بترجع data مباشرة ❌  
دي بترجع **Observable**

👉 يعني: "هجيب الداتا بعد شوية"   Eager Loading 

---

# ⚠️ مهم جداً

الـ Observable **مش بيشتغل غير لما تعمل subscribe**

```ts
this.userService.getUsers(); // ❌ مش هيعمل حاجة

this.userService.getUsers().subscribe(); // ✅ كدا اشتغل
```

---

# 🔥 subscribe (أهم نقطة)

```ts
.subscribe({
  next: (data) => {},
  error: (err) => {},
  complete: () => {}
})
```

## 📌 شرحهم:

### ✅ next

بيتنده لما الداتا تيجي

```ts
next: (res) => {
  console.log(res);
}
```

---

### ❌ error

لو حصل مشكلة في API

```ts
error: (err) => {
  console.log('Error:', err);
}
```

---

### ✅ complete

لما الـ Observable يخلص

```ts
complete: () => {
  console.log('Done');
}
```

---

# 🧱 نبدأ مشروع حقيقي 🔥

## 1. Model

```ts
export interface User {
  id?: number;
  name: string;
  email: string;
  password: string;
  age: number;
}
```

---

# 🧠 2. Service (Professional)

```ts
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { User } from '../models/user';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class UserService {

  private api = 'https://jsonplaceholder.typicode.com/users';

  constructor(private http: HttpClient) {}

  // ✅ GET
  getUsers(): Observable<User[]> {
    return this.http.get<User[]>(this.api);
  }

  // ✅ POST
  addUser(user: User): Observable<User> {
    return this.http.post<User>(this.api, user);
  }

  // ✅ DELETE
  deleteUser(id: number): Observable<any> {
    return this.http.delete(`${this.api}/${id}`);
  }

  // ✅ UPDATE
  updateUser(id: number, user: User): Observable<User> {
    return this.http.put<User>(`${this.api}/${id}`, user);
  }
}
```

---

# 🧾 3. Component + Reactive Form

```ts
import { Component } from '@angular/core';
import { FormGroup, FormControl } from '@angular/forms';
import { UserService } from '../services/user.service';
import { User } from '../models/user';

@Component({
  selector: 'app-user',
  templateUrl: './user.component.html'
})
export class UserComponent {

  users: User[] = [];

  constructor(private userService: UserService) {}

  registerForm = new FormGroup({
    name: new FormControl(''),
    email: new FormControl(''),
    password: new FormControl(''),
    age: new FormControl('')
  });

  // 🔥 GET WITH OBSERVABLE
  loadUsers() {
    this.userService.getUsers().subscribe({
      next: (res) => {
        this.users = res;
        console.log('Data loaded', res);
      },
      error: (err) => {
        console.log('Error loading users', err);
      },
      complete: () => {
        console.log('Finished loading');
      }
    });
  }

  // 🔥 POST
  addUser() {
    const userData = this.registerForm.value as User;

    this.userService.addUser(userData).subscribe({
      next: (res) => {
        console.log('User Added', res);
        this.loadUsers();
      },
      error: (err) => {
        console.log('Error adding user', err);
      },
      complete: () => {
        console.log('Add completed');
      }
    });
  }

  // 🔥 DELETE
  deleteUser(id: number) {
    this.userService.deleteUser(id).subscribe({
      next: () => {
        console.log('Deleted');
        this.loadUsers();
      },
      error: (err) => {
        console.log('Error deleting', err);
      },
      complete: () => {
        console.log('Delete done');
      }
    });
  }

  // 🔥 UPDATE
  updateUser(id: number) {
    const updatedData = this.registerForm.value as User;

    this.userService.updateUser(id, updatedData).subscribe({
      next: (res) => {
        console.log('Updated', res);
        this.loadUsers();
      },
      error: (err) => {
        console.log('Error updating', err);
      },
      complete: () => {
        console.log('Update done');
      }
    });
  }
}
```

---

# 🖼️ 4. HTML

```html
<form [formGroup]="registerForm" (ngSubmit)="addUser()">

  <input type="text" formControlName="name" placeholder="Name">
  <input type="email" formControlName="email" placeholder="Email">
  <input type="password" formControlName="password" placeholder="Password">
  <input type="number" formControlName="age" placeholder="Age">

  <button type="submit">Add</button>
</form>

<hr>

<button (click)="loadUsers()">Load Users</button>

<ul>
  <li *ngFor="let user of users">
    {{ user.name }} - {{ user.email }}

    <button (click)="deleteUser(user.id!)">Delete</button>
    <button (click)="updateUser(user.id!)">Update</button>
  </li>
</ul>
```

---

# 🔥 الفرق بين Observable و Promise (مهم جداً)

|Observable|Promise|
|---|---|
|بيبعت أكتر من value|مرة واحدة بس|
|lazy (لازم subscribe)|بيشتغل فوراً|
|فيه next/error/complete|فيه then/catch بس|

---

# 🧠 Flow المشروع الحقيقي

1. الفورم ياخد input
    
2. تضغط submit
    
3. تاخد `.value`
    
4. تبعتها للـ Service
    
5. Service تضرب API
    
6. يرجع Observable
    
7. تعمل subscribe
    
8. تتعامل مع:
    
    - next → الداتا
        
    - error → المشكلة
        
    - complete → النهاية
        

---

# 💥 مستوى أعلى شوية (Pro Tips)

### ✅ استخدم async pipe بدل subscribe (بعد كدا)

### ✅ اعمل error handling global

### ✅ استخدم interceptors

---

# 🚀 لو عايز نكمل

نقدر نعمل مع بعض:

- login system (JWT 🔥)
    
- loading spinner
    
- toast notifications
    
- real backend بـ Spring Boot
    

---

لو في نقطة لسه مش واضحة (خصوصاً Observable 😈) قولي وهنفجرها تبقى أسهل من المية 💯