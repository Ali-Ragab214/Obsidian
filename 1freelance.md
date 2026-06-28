## الخطوات كاملة

---

### 1. Clone الـ Repo مش Fork

```bash
# Fork = نسخة منفصلة باسمك
# Clone = تاخد الكود على جهازك وتشتغل على نفس الـ repo

git clone https://github.com/FRIEND_USERNAME/REPO_NAME.git
cd REPO_NAME
```

---

### 2. الـ .env file

```bash
# حطه في الـ root بتاع المشروع
# وتأكد إنه في الـ .gitignore
cat .gitignore | grep .env
```

لو مش موجود في الـ .gitignore أضفه:

```bash
echo ".env" >> .gitignore
```

---

### 3. مش بتبوش على الـ main مباشرة

```bash
# دايماً اعمل branch جديدة
git checkout -b feature/your-feature-name

# اشتغل وبعدين
git add .
git commit -m "feat: your feature"
git push origin feature/your-feature-name
```

---

### 4. بعدين اعمل Pull Request

```
على GitHub → New Pull Request
من branch بتاعك → main
وصاحبك يعمل Review ويـ Merge
```

---

### الـ Flow كامل

```
clone → اعمل branch → اشتغل → push → PR → صاحبك يـ merge
```

> **مش بتبوش على main أبداً** — دي الـ protected branch. 🎯