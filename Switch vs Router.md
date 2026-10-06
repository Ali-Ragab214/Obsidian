آه طبعًا. خلينا نفهمها **من الصفر وبطريقة CCNA**، والأهم نفرّق بينهم من ناحية: _كل جهاز بيعمل إيه؟ والـ packet بتتحرك إزاي؟_

### أولًا: شكل الأجهزة في الحقيقة

**Switch** غالبًا هتلاقيه بالشكل ده: جهاز عليه عدد كبير من Ethernet ports، وبيوصل أجهزة الشبكة ببعض.

![Image](https://images.openai.com/static-rsc-4/wlfTRYrtnD9AKI-qCfI1XVRrZu4noEOvITEwlt87R_rUFUGOfaH92_zZIXsRJ98NVhzuzRy6V-T2PbAJvNwmGnbBXz3AVlfBtM5GNCVowfZkM-Pl4UyKBY1VD8pd4MOCT-UlYyKaw8v6J9oEyO8KUDEWrrrDxUvbtpn-ddt422HQLiuxnFfcL2wNMr9-KbLv?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/lxao_oxaBUDW12MqtoutwjXLtCQ6tCPfBduWSAoGj-lpnGlCta8KQvj8n2M0O0q2UWgbefxaFefl25XH1-LipeNOlybq_glvA32MUcYAIOtINPL7ddpWS7c6jzR3JQsfQ5loXAnqmkUz0iGQzsmFCUbmd4JdaQ8WLU5R3fEtphiP8eQLKdmqAmrUO0LYx7Zg?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/aKfY3o5HTKQf168gdSmINoF-lNOTWgyqKVWhMxyBG6d5psvR-0taLW-6yAng_7CDl5HtikbhNFz2AIb6q3HW1_vFonoVoTqoY5KNoBLa7BZPwwOvjer3TULY4QDHSmA-5TdEt0kxJwz4gf8GADjx5HXbdcnr_UTsAQdi-yw4Kx-s4kifnXLs4inEjrXeIzUY?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/pWyOCATxLQoHyT2ttsWxCsO6kmjDBA0AJIO-iJ1n_JXCeQsCcK0pNvZNT053XNckgOKkA9E8IXnKaAKGj7IVr2VLludyHgfqaoGuKur7Erew6YakIrQosZU9O0OKQTUtxBT1T2i3LkMdlBBr9tNN-HhZfH-hlpPnJCg4JFMdCS_NHTKdfOL_X3nNZt8zUN5u?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/wiBCSPhXMtwYyLW-0hGHs_bxOtPyEwgA0q6CLI2g0Lw7317cnhBw9SOjXRHtPWvDPpA9_T2dPsTcWONvjzeGQC-VQu7X7W-bKQgFhHIicEpfTRp5TQQgYkpIexj64JsR9jUYFSB326YgC2eIrmyLx3SHTZFAtvcodQX6_V0ukbSdSfjxd6hxZ9J28VCLV_IA?purpose=fullsize)

**Router** برضه جهاز Networking، لكن وظيفته الأساسية مختلفة: بيربط **شبكات مختلفة** ببعض.

![Image](https://images.openai.com/static-rsc-4/kas5PDThKTzpxMJl95SzsbAlRuqiE_D30oz38mL8Beldo5M7UuvQsKW4xtgyHVxqP3pwQ8Ncwi-29Swwbt_ghgtmF_ztNEWt8hmlYPe-AVyuTk2dlnS2z0E6HunVuR3CSg0W3qEFI5pLV4mH_XHFu-RyRb15jCkogo1JaDkDWiLdECXk-uoHmFGwnI2HUYnU?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/gdQSYChLdLc70gC_KPnDaO_IhR7v9vns2WC5JcvVxiiVb2lqyA-LIlfI20DnalRS6_IcT-yLw1d6jQSEzWW9SAD9Umt8-7hl9iVlH6DRsIsdWZUslrgXe5mgwc63dSGCyVhPqA6jRz3H8ivgCyx7NFN91HyCVk9FfXrz53_8tywlwvbw1Tvpimw4NzHRyd6N?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/2G8PqO7u_41qNBJFa0n0FGOOQGBUpXrly7gOB0X_sMAe3tytkif203dmw29WwNYXEMUx56oVzxdk8cIglw22vXC2vc91G2N0zX-dZ86GsxT5bGN4Loe3ueOZ2dW1PaG-POAuqeFvjeV3u2f3abB7xTkhl9ODkqHXvpG7y2ukh0r-mh_wcZI7Hu13arjE7KBU?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/j41hYbebQPr9TerSS1wf2rjlek6HEEV388jzLTPkuSRY4rI6rvRJR6Jm14jcUDs_GEDZOSR8o5CT66FAjLbNijLxSmcsDLWNyHKYbOTYHzguYzGVCJPrT7G27xDcMe5RvXB_fOUCmZbr-TwZaB9xzYMeSwW7LvEPjezDm0K6sVy6vy_cWwwPXb3BzDzUZZuf?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/pWaUUKY5k-7Fu0X7glVMC02_T2UC7icv7Qzfo8_NHwTm4tZ4DFl3pq1svDtmMnTTtCA1ruOgP4SifY7OAo5tWhNHT2VWEgKlYh-P7xaDw2-8-Et7QH4sL4NCn01OE6ULNSN8uqukKbQHM5rhDfdynZJEf3FD4pN7kkOb_bmYSe_tic7SV_cMxXKtf8IE2Q4y?purpose=fullsize)

> ملحوظة: في الـ enterprise ممكن شكل الـ Router والـ Switch يبقى متشابه جدًا، فـ **ما تعتمدش على الشكل** في التفرقة.

---

# 1. الـ Switch بيعمل إيه؟

تخيل عندك شركة صغيرة:

```text
PC 1 ───┐
PC 2 ───┤
PC 3 ───┤
Printer ┤─── SWITCH
Server ─┤
AP ─────┘
```

الـ **Switch** وظيفته الأساسية إنه يربط الأجهزة الموجودة في **نفس الشبكة المحلية LAN**.

يعني مثلًا:

```text
192.168.1.10  PC1
192.168.1.20  PC2
192.168.1.30  PC3
```

كلهم في نفس الشبكة:

```text
192.168.1.0/24
```

والـ Switch يخليهم يتواصلوا مع بعض.

---

# 2. طيب الـ Switch بيعرف يبعت البيانات لمين إزاي؟

هنا ندخل في أهم نقطة.

الـ Switch بيشتغل أساسًا باستخدام **MAC Address**.

مثلًا:

```text
PC1
IP  = 192.168.1.10
MAC = AA:AA:AA:AA:AA:AA

PC2
IP  = 192.168.1.20
MAC = BB:BB:BB:BB:BB:BB
```

الـ Switch بيتعلم:

```text
MAC Address              Port

AA:AA:AA:AA:AA:AA   →    Port 1
BB:BB:BB:BB:BB:BB   →    Port 2
CC:CC:CC:CC:CC:CC   →    Port 3
```

ودي اسمها **MAC Address Table**.

فلو PC1 عايز يبعت لـ PC2:

```text
PC1
 │
 │ Ethernet Frame
 │ Destination MAC = BB:BB:BB:BB:BB:BB
 ↓
SWITCH
 │
 │ "آه، الـ MAC ده عند Port 2"
 ↓
PC2
```

يعني ببساطة:

> **Switch → بيفكر بالـ MAC Address وبيعمل Switching للـ Frames.**

---

# 3. طيب الـ Router؟

الـ Router بيظهر لما يكون عندك **شبكات مختلفة**.

مثلًا:

```text
LAN 1                         LAN 2

192.168.1.0/24                192.168.2.0/24

PC1 ── Switch ── Router ── Switch ── PC2
```

هنا:

```text
PC1 = 192.168.1.10

PC2 = 192.168.2.10
```

دول **مش في نفس الـ network**.

الـ Switch لوحده مش هو المسؤول عن توصيلهم كشبكتين مختلفتين.

هنا الـ Router بييجي.

---

# 4. الـ Router بيعمل إيه بالظبط؟

الـ Router بيعمل **Routing** بين الشبكات.

مثلاً عنده:

```text
Router

Interface G0/0
IP = 192.168.1.1

Interface G0/1
IP = 192.168.2.1
```

فيبقى:

```text
192.168.1.0/24
       │
       │
   [ ROUTER ]
       │
       │
192.168.2.0/24
```

فالـ Router عارف:

```text
Destination Network      Interface

192.168.1.0/24       →   G0/0
192.168.2.0/24       →   G0/1
```

ودي موجودة في **Routing Table**.

---

# 5. أهم فرق تحفظه

||Switch|Router|
|---|---|---|
|الوظيفة الأساسية|يربط الأجهزة داخل LAN|يربط Networks مختلفة|
|يعتمد أساسًا على|MAC Address|IP Address|
|البيانات التي يتعامل معها|Frames|Packets|
|الجدول الأساسي|MAC Address Table|Routing Table|
|OSI|Layer 2|Layer 3|
|مثال|PC ↔ PC|LAN ↔ Internet|

ودي من أهم الحاجات في Networking:

```text
SWITCH
Layer 2
   ↓
MAC Address
   ↓
Frame
```

مقابل:

```text
ROUTER
Layer 3
   ↓
IP Address
   ↓
Packet
```

---

# 6. طب نشوفهم مع بعض في Network حقيقية

ده الشكل اللي عايزك تتخيله:

![Image](https://images.openai.com/static-rsc-4/OoEw-9qbgEHCRQPScvKglx5z4bbjs6lAtKHc2djaqa7B3mItbHW-MMTFy5eNpUAo_PlEsclIPkSw82XqKOTeuhWPymYTd4Zt3AyCqeE6RFVzOeyP6k7Z5WRNGRDSQtGXaHiAnpIlgFSydweurnpovLsXSeMPv2q-Ga2EspGeCk_LBYJipuqrt-yPUkdONRqa?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/AS9KswN6_oC0mfZesgCPgygNBXQ59WrauTuMaIZ47h60YZDB9g3HCx2tNRIY1cRxX1mDuzc8hPa2oD59eTwV6Rd2HgIiR91ODf8zJHow0pZaqn9tcpfIpdK4kXIa_JqoMpei3qH6fqKFgHhj-JgljgYvttR5JIF0HTZANzGf8eV53EsOER7Jd88MnVTdb-nw?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/vHxjAd5HU4xKVpSRj6aHjO9AlKa3VXFZUIvCxzyAGUgY74CGt1mJdLYK9alx_VgPG4nHfqgM9V-sv4wfhmABSsyaUuP8iN87ULEmNvnmZt4q4bVRYia3aOieYjBwX8c35fygleRaMxbbSn1KDGL_80M5LolO6vIUCsRHz3_VQ3qNcdEtFSEFqhiwatvhKTWz?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/xPpBpLpqt7I10dFwoMMbKMU2bfYyPvCXVD1gqmwunWfoKipstNpAAdtquMqZgCCdr9hPlUkv21Kf3sNXzGBB8duWFG4ftV6NkX33fHuTdgqO_aUtLW_p1fDIe4YxPlowaJu783P11c-gEl7vz-j4pj8QlFFQsmDvpEnUINH2-e1rGOO9fXM_85FmyivGSWjK?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/N8OXeVAR9NG88FKAOgyCOQreIS95Fd82zpkhwVfmg60NTBOiMogM3p90ggO4ha-ha1hP8FqgC29ZhElklP2fTboi77bpXiXyAJu8vvCUxzXZCh7Ti4uyfJ9jahXTZtrZGhPSasOqatOsvP6iZl0h5u5Jq741xTym8eo5G6Nn8yovnzsq8ToWhX4o-l0pxczY?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/jf7tPlqbMi268zrpDctNJJtpAtdrunfurSoFaQegCmR1WHQrhzmQwaqNHJJNPzw1_i7-xYDuNLhMg8ADbQn__VHGVLnFk9dy7QNwNeW-H7CHqt-QHhGAQPqYKnOJwmilf1eN0a60Iz7SYTn9rmrXAR_T2zM6Ji8NxpMCM9M-2p9-tSDjm2U2t5wA3mEAnkTi?purpose=fullsize)

مثلًا:

```text
                    INTERNET
                       │
                       │
                   [ ROUTER ]
                       │
                 192.168.1.1
                       │
                    [SWITCH]
                 ┌─────┼─────┐
                 │     │     │
                PC1   PC2   PC3
```

هنا:

### Switch

مسؤول عن:

```text
PC1
PC2  ───→ Switch
PC3
```

يعني **الأجهزة داخل الـ LAN**.

### Router

مسؤول عن:

```text
LAN
 │
 ↓
Router
 │
 ↓
Internet
```

يعني **الخروج من الشبكة لشبكة أخرى**.

---

# 7. مثال من بيتك 🏠

الـ Home Router اللي عندك غالبًا بيعمل **أكتر من وظيفة في جهاز واحد**.

مثلًا:

```text
                 INTERNET
                    │
                    │
               [ HOME ROUTER ]
                /     |      \
               /      |       \
             PC     Phone     TV
```

الجهاز ده غالبًا جواه:

```text
Router
   +
Switch
   +
Wi-Fi Access Point
   +
NAT
   +
DHCP
   +
Firewall
```

عشان كده الناس بتقول عليه ببساطة **Router**.

لكن فعليًا الجهاز multifunctional.

---

# 8. أهم سيناريو تفهمه

افترض:

```text
PC1
IP = 192.168.1.10

PC2
IP = 192.168.1.20
```

نفس الـ network:

```text
192.168.1.0/24
```

PC1 عايز يكلم PC2:

```text
PC1
 ↓
Switch
 ↓
PC2
```

**الـ Router مش محتاج يدخل في الموضوع.**

لكن لو:

```text
PC1
192.168.1.10

Google Server
8.8.8.8
```

هنا:

```text
PC1
 ↓
Switch
 ↓
Router
 ↓
Internet
 ↓
Google
```

لأن `8.8.8.8` موجود في **شبكة مختلفة**.

---

## 🧠 الخلاصة اللي تحفظها

افتكرها كده:

> **Switch = داخل الشبكة**  
> **Router = بين الشبكات**

أو:

```text
           SAME NETWORK
               
 PC ────┐
 PC ────┼──── SWITCH
 PC ────┘
```

لكن:

```text
        NETWORK A
            │
         SWITCH
            │
         ROUTER
            │
         SWITCH
            │
        NETWORK B
```

**Switch يسأل:**

> "الـ MAC Address ده موجود على أنهي Port؟"

**Router يسأل:**

> "الـ Destination IP ده موجود في أنهي Network؟ وأطلّع الـ packet من أنهي interface؟"

وده بالضبط الأساس اللي بعده هنفهم منه **IP Addressing → Subnetting → ARP → Default Gateway → Routing Table**.