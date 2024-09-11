---
date: 2024-09-11T17:27:51+08:00
title: POSIX 线程库中线程安全和线程同步的函数
description: POSIX 线程库中线程安全和线程同步的函数
featured_image: /images/book.png
images:
  - /images/book.png
tags:
  - tutorial
  - C
categories: Tutorials
---
以下是POSIX 线程库（pthreads）中常用的几类线程安全相关函数，包括互斥锁、条件变量、读写锁和线程安全属性等。

### 1. 互斥锁 (Mutex)

互斥锁用于在多个线程之间实现互斥访问共享资源。互斥锁的基本操作包括初始化、加锁、解锁和销毁。

- **`pthread_mutex_init`**: 初始化互斥锁。

    ```c
    int pthread_mutex_init(pthread_mutex_t *mutex, const pthread_mutexattr_t *attr);
    ```

    - `mutex`: 指向要初始化的互斥锁。
    - `attr`: 互斥锁的属性，可以为 `NULL` 表示默认属性。
    - **返回值**: 成功返回 `0`，失败返回错误码。

- **`pthread_mutex_lock`**: 加锁操作，使调用线程获得互斥锁。如果锁已经被其他线程持有，则调用线程会阻塞，直到锁可用。

    ```c
    int pthread_mutex_lock(pthread_mutex_t *mutex);
    ```

    - `mutex`: 指向要锁住的互斥锁。
    - **返回值**: 成功返回 `0`，失败返回错误码。

- **`pthread_mutex_trylock`**: 尝试加锁。如果锁不可用（已被其他线程持有），不会阻塞调用线程，而是立即返回。

    ```c
    int pthread_mutex_trylock(pthread_mutex_t *mutex);
    ```

    - **返回值**: 成功返回 `0`，如果锁不可用返回 `EBUSY`。

- **`pthread_mutex_unlock`**: 解锁操作，释放调用线程持有的互斥锁。

    ```c
    int pthread_mutex_unlock(pthread_mutex_t *mutex);
    ```

    - **返回值**: 成功返回 `0`，失败返回错误码。

- **`pthread_mutex_destroy`**: 销毁互斥锁，释放其资源。

    ```c
    int pthread_mutex_destroy(pthread_mutex_t *mutex);
    ```

    - **返回值**: 成功返回 `0`，失败返回错误码。

### 2. 条件变量 (Condition Variable)

条件变量允许线程阻塞，等待某个条件的发生。

- pthread_cond_init`**: 用于初始化一个条件变量。

    ```c
    int pthread_cond_init(pthread_cond_t *cond, const pthread_condattr_t *attr);
    ```

	 - `cond`: 要初始化的条件变量的指针。
	 - `attr`: 条件变量的属性，可以为 `NULL` 表示使用默认属性。
	 - **返回值**: 成功返回 `0`，失败返回错误码。

- pthread_cond_wait`**: 使调用线程阻塞，等待特定条件的发生。当线程调用此函数时，它会自动释放与之关联的互斥锁，并将线程置于等待条件变量的状态。此函数返回时，调用线程将重新获得互斥锁。

    ```c
    int pthread_cond_wait(pthread_cond_t *cond, pthread_mutex_t *mutex);
    ```

    - `cond`: 条件变量指针。
    - `mutex`: 指向互斥锁的指针，该锁必须在调用 `pthread_cond_wait` 之前被锁住。
    - **返回值**: 成功返回 `0`，失败返回错误码。

- **`pthread_cond_signal`**: 唤醒等待条件变量的一个线程。

    ```c
    int pthread_cond_signal(pthread_cond_t *cond);
    ```

    - `cond`: 条件变量指针。
    - **返回值**: 成功返回 `0`，失败返回错误码。

- **`pthread_cond_broadcast`**: 唤醒等待条件变量的所有线程。

    ```c
    int pthread_cond_broadcast(pthread_cond_t *cond);
    ```

    - `cond`: 条件变量指针。
    - **返回值**: 成功返回 `0`，失败返回错误码。

- **`pthread_cond_destroy`**: 销毁条件变量，释放其资源。

    ```c
    int pthread_cond_destroy(pthread_cond_t *cond);
    ```

    - `cond`: 条件变量指针。
    - **返回值**: 成功返回 `0`，失败返回错误码。

### 3. 读写锁 (Read-Write Lock)

读写锁允许多个线程同时读取共享数据，但写操作需要独占锁。适用于读多写少的场景。

- **`pthread_rwlock_init`**: 初始化读写锁。

    ```c
    int pthread_rwlock_init(pthread_rwlock_t *rwlock, const pthread_rwlockattr_t *attr);
    ```

    - `rwlock`: 读写锁指针。
    - `attr`: 读写锁属性，可以为 `NULL` 表示默认属性。
    - **返回值**: 成功返回 `0`，失败返回错误码。

- **`pthread_rwlock_rdlock`**: 获取读锁。如果写锁已经被占用，调用线程将被阻塞。

    ```c
    int pthread_rwlock_rdlock(pthread_rwlock_t *rwlock);
    ```

    - **返回值**: 成功返回 `0`，失败返回错误码。

- **`pthread_rwlock_wrlock`**: 获取写锁。如果读锁或写锁已经被占用，调用线程将被阻塞。

    ```c
    int pthread_rwlock_wrlock(pthread_rwlock_t *rwlock);
    ```

    - **返回值**: 成功返回 `0`，失败返回错误码。

- **`pthread_rwlock_unlock`**: 释放读锁或写锁。

    ```c
    int pthread_rwlock_unlock(pthread_rwlock_t *rwlock);
    ```

    - **返回值**: 成功返回 `0`，失败返回错误码。

- **`pthread_rwlock_destroy`**: 销毁读写锁，释放其资源。

    ```c
    int pthread_rwlock_destroy(pthread_rwlock_t *rwlock);
    ```

    - **返回值**: 成功返回 `0`，失败返回错误码。

### 4. 信号量 (Semaphore)

信号量提供一种计数机制，用于控制多线程对共享资源的访问。

- **`sem_init`**: 初始化一个信号量。

    ```c
    int sem_init(sem_t *sem, int pshared, unsigned int value);
    ```

    - `sem`: 信号量指针。
    - `pshared`: 如果为 `0`，表示信号量用于当前进程的线程间通信；如果为非零，表示用于进程间通信。
    - `value`: 信号量的初始值。
    - **返回值**: 成功返回 `0`，失败返回 `-1`。

- **`sem_wait`**: 减少信号量的值。如果信号量的值为 `0`，调用线程将被阻塞，直到信号量的值大于 `0`。

    ```c
    int sem_wait(sem_t *sem);
    ```

    - **返回值**: 成功返回 `0`，失败返回 `-1`。

- **`sem_post`**: 增加信号量的值。如果有线程在等待信号量，它将被唤醒。

    ```c
    int sem_post(sem_t *sem);
    ```

    - **返回值**: 成功返回 `0`，失败返回 `-1`。

- **`sem_destroy`**: 销毁信号量，释放其资源。

    ```c
    int sem_destroy(sem_t *sem);
    ```

    - **返回值**: 成功返回 `0`，失败返回 `-1`。

### 5. 屏障 (Barrier)

屏障用于使一组线程在到达同步点前保持阻塞，直到所有线程都到达这个点。

- **`pthread_barrier_init`**: 初始化一个屏障对象。

    ```c
    int pthread_barrier_init(pthread_barrier_t *barrier, const pthread_barrierattr_t *attr, unsigned int count);
    ```

    - `barrier`: 屏障对象指针。
    - `attr`: 屏障属性，可以为 `NULL`。
    - `count`: 要同步的线程数量。
    - **返回值**: 成功返回 `0`，失败返回错误码。

- **`pthread_barrier_wait`**: 使调用线程在屏障上等待，直到指定数量的线程都调用了该函数。

    ```c
    int pthread_barrier_wait(pthread_barrier_t *barrier);
    ```

    - **返回值**: 所有线程都到达时返回 `PTHREAD_BARRIER_SERIAL_THREAD`，否则返回 `0`。

- **`pthread_barrier_destroy`**: 销毁屏障对象。

    ```c
    int pthread_barrier_destroy(pthread_barrier_t *barrier);
    ```

    - **返回值**: 成功返回 `0`，失败返回错误码。

### 6. 自旋锁 (Spinlock)

自旋锁是另一种类型的锁，适用于持锁时间很短的情况。与互斥锁不同，自旋锁不会引起调用线程休眠，而是不断“自旋”，直到锁可用。

- **`pthread_spin_init`**: 初始化自旋锁。

    ```c
    int pthread_spin_init(pthread_spinlock_t *lock, int pshared);
    ```

    - `lock`: 自旋锁指针。
    - `pshared`: 共享标志（0表示仅用于同一进程中的线程）。
    - **返回值**: 成功返回 `0`，失败返回错误码。

- **`pthread_spin_lock`**: 获取自旋锁。如果锁不可用，调用线程会忙等待。

    ```c
    int pthread_spin_lock(pthread_spinlock_t *lock);
    ```

    - **返回值**: 成功返回 `0`，失败返回错误码。

- **`pthread_spin_unlock`**: 释放自旋锁。

    ```c
    int pthread_spin_unlock(pthread_spinlock_t *lock);
    ```

    - **

返回值**: 成功返回 `0`，失败返回错误码。

- **`pthread_spin_destroy`**: 销毁自旋锁，释放其资源。

    ```c
    int pthread_spin_destroy(pthread_spinlock_t *lock);
    ```

    - **返回值**: 成功返回 `0`，失败返回错误码。
