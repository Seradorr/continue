---
name: Production Quality Rules
description: |
  Uretim seviyesi kalite, guvenlik ve verification bari.
alwaysApply: true
---

# PRODUCTION QUALITY RULES

- Etkilenen callers, interfaces, configs, scripts ve operasyonel akislar dusunulmeden degisiklik tam sayilmaz.
- Secret, token, credential, certificate ve private data'yi gereksiz yere ciktiya tasima.
- Refactor veya fix sonrasi uygun verification adimini belirt.
- Davranis degisikligi varsa backward-compat ve rollout riskini kisaca dusun.
- Istenen is bittikten sonra ancak kanit varsa en fazla 3 iyilestirme onerisi sun.
