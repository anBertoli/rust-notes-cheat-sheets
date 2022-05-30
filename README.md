## Interior mutability			
Interior mutability is the property for which if you have shared references to a wrapper type (eg. &Cell<T>) you can still mutate the value contained in the wrapper (T). It’s useful when you need to introduce mutability inside of something immutable or when you need a mutable part of a data structure, but still logically present the structure as immutable. In other words, we can have an immutable value or multiple immutable references to a value, but still mutate its content. Mutation is performed in controlled and safe ways, depending on the wrapper type.

### `Cell<T>`

Interior mutability for Copy types via copies. Setting a value means putting inside the Cell a copy of the value to set. Getting a value means obtaining a copy of the wrapped value. You can never obtain a pointer to the value inside the Cell.


| Provides | Accessors | Panics| Send | Sync |
|:---:|:---:|:---:|:---:|:---:|
| Values (copies) | `.get()` `.set()` <br><sub>to get/set a copy</sub> | Never | ✅<br><sub>(if T is Send)</sub> | 🚫 |

#### Safety Notes
<p>1) No references to the inner value can be obtained. There's no risk to mutate the value while someone is holding a pointer to the inner value. 2) Cell is not Sync (no &Cell can be shared between threads) because getting/setting the value is not synchronized. 3) Cell is Send if T is Send: if T is Send there's no problem in moving the Cell and using it at different times. If T is not Send and Cell was Send nonetheless, T could end up being used in different threads, invalidating the Send safety limit imposed on it.</p>