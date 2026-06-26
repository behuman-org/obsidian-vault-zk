---
tags:
  - concepto
---

Información personal identificable: tu nombre, número de documento, foto, fecha de nacimiento... cualquier dato que permita saber _quién sos_. 
El punto clave del diseño es que **el PII nunca toca la cadena**.
El issuer lo ve para verificarte, pero lo que sale hacia el sistema es solo una credencial y un _commitment_ (un hash, una huella digital irreversible del secreto de tu identidad). Así se logra la tensión "real + anónimo": alguien confirmó que sos real, pero on-chain nadie ve tus datos.