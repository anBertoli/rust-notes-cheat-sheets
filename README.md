## Interior mutability			
Interior mutability is the property for which if you have shared references to a wrapper type (eg. &Cell<T>) you can still mutate the value contained in the wrapper (T). It’s useful when you need to introduce mutability inside of something immutable or when you need a mutable part of a data structure, but still logically present the structure as immutable. In other words, we can have an immutable value or multiple immutable references to a value, but still mutate its content. Mutation is performed in controlled and safe ways, depending on the wrapper type.

### `Cell<T>`

Interior mutability for Copy types via copies. Setting a value means putting inside the Cell a copy of the value to set. Getting a value means obtaining a copy of the wrapped value. You can never obtain a pointer to the value inside the Cell.

| Type | Provides | Accessors | Panics| Send | Sync |
|:---:|:---:|:---:|:---:|:---:|:---:|
| `Cell<T>` | Values (copies) | `.get()` <br>`.set()` <br><sub>to get/set a copy</sub> | Never | ✅<br><sub>(if T is Send)</sub> | 🚫 |

#### Safety Notes
<p>1) No references to the inner value can be obtained. There's no risk to mutate the value while someone is holding a pointer to the inner value. 2) Cell is not Sync (no &Cell can be shared between threads) because getting/setting the value is not synchronized. 3) Cell is Send if T is Send: if T is Send there's no problem in moving the Cell and using it at different times. If T is not Send and Cell was Send nonetheless, T could end up being used in different threads, invalidating the Send safety limit imposed on it.</p>


### `RefCell<T>`

Interior mutability for all types via references to the inner value. RefCell allows you to borrow the wrapped value either mutably or immutably. Dynamic run-time borrowing ensures no more than one mutable borrow or mixed borrows occur at the same time (mixed = both & and &mut at the same time).

When the inner value is borrowed, a Ref or RefMut is returned, which can be used as a reference to the inner value. When this object is dropped, the internal borrowing bookeeping is reverted accordingly. These syntethic references point to the original RefCell, so the RefCell cannot be moved/dropped until all these refs are dropped.

| Type | Provides | Accessors | Panics| Send | Sync |
|:---:|:---:|:---:|:---:|:---:|:---:|
| `RefCell<T>` | References (&/&mut) | `.borrow()` `.borrow_mut()` <br><sub>to get the Ref/RefMut</sub> <br><br> `.deref()` `.deref_mut()`<br> <sub>on the Ref/RefMut</sub> | Mixed borrows or more than one mutable borrow | ✅<br><sub>(if T is Send)</sub> | 🚫 |

#### Safety Notes
<p>1) Returned references (Ref/RefMut) are checked via dynamic borrowing. There is no way we can obtain more than one exclusive ref or mixed refs to the inner value at the same moment. 2) RefCell is not Sync (no &RefCell can be shared between threads) because updates of the internal borrowing state are not synchronized. 3) RefCell is Send if T is Send: if T is Send there's no problem in moving the RefCell and using it at different times. If T is not Send and RefCell was Send nonetheless, T could end up being used in different threads, invalidating the Send safety limit imposed on them.</p>


### `Mutex<T>`

A mutual exclusion primitive useful to protect data shared across threads. The Mutex provides interior mutability via references in a thread safe way, since the access to the inner value is properly synchronized. We can compare Mutex to RefCell because both provide a similar dynamic run-time borrowing, but Mutex blocks the thread waiting for the lock instead of panicking.

The internal data can be accessed via the lock method, which returns a MutexGuard. This guard can be treated like a pointer to the inner value. Holding a guard is a proof that the inner data is being accessed only by the (unique) holder of the guard. When the guard is dropped, other lock() calls can access the inner value. MutexGuards has a lifetime >= of the original Mutex, so the mutex cannot be moved/dropped until all guards are dropped.

| Type | Provides | Accessors | Panics| Send | Sync |
|:---:|:---:|:---:|:---:|:---:|:---:|
| `Mutex<T>` | References (&/&mut) | `.lock()` `.borrow_mut()` <br><sub>to get the MutexGuard</sub> <br><br> `.deref()` `.deref_mut()`<br> <sub>on the MutexGuard</sub> | Never, blocks until the lock is freed | ✅<br><sub>(if T is Send)</sub> | ✅<br><sub>(if T is Send)</sub> |

#### Safety Notes
<p>1) Returned guards are checked via dynamic borrowing. There is no way we can obtain more than one guard at the same moment. 2) Mutex is Send + Sync only if the internal T is Send: if T is Send there's no problem in moving the Mutex and using it at different times (because T is itself Send and can be used in different threads at different times safely). If T is not Send, T could end up being used in different threads, invalidating the Send safety limit imposed on them. 3) Sync on T is not influent: the data is accessed from one thread at a time in any case.</p>




