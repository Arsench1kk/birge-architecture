---
id: ADR-004
status: Accepted
date: 2026-04-22
deciders: [arsen]
---

# ADR-004 — JWT хранение: Keychain, не UserDefaults

## Context
Access token живёт 60 минут, refresh token — 30 дней.
Нужно безопасное персистентное хранилище между сессиями.

## Decision
Refresh token → iOS Keychain (kSecAttrAccessibleWhenUnlocked).
Access token → в памяти (State в AppFeature) — НЕ персистируется.

## Rationale
Keychain шифруется hardware Secure Enclave на iPhone 12+.
UserDefaults — plaintext на диске, уязвим при джейлбрейке.
Access token в памяти не нужен после рестарта — делаем refresh.

## Consequences
- KeychainClient зарегистрирован как TCA Dependency → полностью мокируемый
- При force-quit и рестарте: AppFeature проверяет Keychain → если есть refreshToken → сразу .authenticated
- При logout: удалить из Keychain + обнулить State

## Related
[[Architecture/Security_and_Authentication]] Section 1
[[Tasks/iOS_Sprint_1]] IOS-003
