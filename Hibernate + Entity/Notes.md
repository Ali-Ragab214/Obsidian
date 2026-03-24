## - (Code)

@Id  
@GeneratedValue(strategy = GenerationType.AUTO)

❓ الفرق بين AUTO و IDENTITY؟

✅ **الإجابة:**

- IDENTITY → DB Auto Increment
- AUTO → Hibernate يختار strategy


## (Concept)

اشرح الفرق بين:

- Transient
- Persistent
- Detached

✅ **الإجابة:**

- Transient → object مش في DB
- Persistent → مربوط بالـ session
- Detached → خرج من session