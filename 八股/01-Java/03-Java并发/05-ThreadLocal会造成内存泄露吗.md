---
aliases:
  - ThreadLocal会造成内存泄露吗
---

# ThreadLocal 会造成内存泄露吗？

> 本篇先建立问题骨架，后续再逐步补充 ThreadLocalMap、弱引用、线程池生命周期和清理机制。

## 一句话回答

ThreadLocal 本身不是必然造成内存泄露，但在线程池等长生命周期线程中，如果使用后不调用 `remove()`，可能留下无法再访问的 value，最终造成内存占用持续增长。

## 它解决什么问题？

## ThreadLocal 的存储位置

## 为什么 key 使用弱引用？

## value 为什么可能残留？

## 为什么线程池场景更危险？

## 如何正确清理？

## 常见追问

## 面试口述版

## 相关笔记

- [[01-Java 内存模型（JMM）是什么？|Java 内存模型（JMM）是什么？]]
- [[03-Synchronized是什么|Synchronized是什么]]
- [[04-ReentrantLock是什么|ReentrantLock是什么]]
