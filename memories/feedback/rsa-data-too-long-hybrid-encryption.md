---
name: RSA报文超长改用混合加密模式
description: 当RSA签名/验签报"数据太长"时，改用AES-GCM加密数据+RSA-OAEP加密AES密钥的混合加密模式，跨项目通用
type: feedback
---

当使用 RSA 签名的明文传输模式（data 字段为明文 JSON）时，如果服务端验签报"数据太长"，原因是 RSA 有块长度限制（2048位密钥 + OAEP 填充最多约190字节），明文 data 超出了限制。

**解决方案：改用混合加密模式（Hybrid Encryption）**

1. 生成随机 AES-128 密钥
2. 用 AES-GCM 加密业务数据（无长度限制）→ data 字段
3. 用 RSA-OAEP 加密 AES 密钥（仅16字节，在 RSA 限制内）→ encKey 字段
4. 签名覆盖 data + encKey（7参数签名，比明文模式多一个 encKey）
5. 服务端：7参数验签 → RSA解密encKey得到原始AES密钥 → AES-GCM解密data得到明文

**关键易混淆点：encKey 不是直接的 AES 密钥，而是 RSA 加密后的 AES 密钥。** 服务端必须先 RSA 解密 encKey，再用解密得到的原始密钥做 AES-GCM 解密。组长描述"使用AES-GCM解密（秘钥为encKey）"是简写，实际是"秘钥从encKey经RSA解密获得"。

**Why:** 2026-07-13 在 open-sdk-java 项目中，unify.scan.barcodepay 接口的 bizContent 字段较多，明文序列化后超出 RSA 块长度限制，银盛方验签报数据太长。组长给出上述混合加密方案。

**How to apply:**
- 任何使用 RSA 签名的支付/通信 SDK，当 data 字段较长时，优先考虑混合加密模式
- 客户端：encryptRequest（AES加密数据 + RSA加密密钥）→ signData（7参数）→ 发送
- 服务端：verifyDataOnly（7参数验签）→ decryptByPrivateKey（RSA解密encKey）→ decryptByAes（AES-GCM解密data）
- 响应方向同理：服务端加密+签名 → 客户端验签+RSA解密encKey+AES-GCM解密data
- 签名原文格式：serialNo\ntimestamp\nappId\ndata\nencKey\n（每个字段后跟换行符，包括最后一个）
