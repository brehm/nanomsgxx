---
layout: docpage
title: 3. Sockets
prev: Messages
next: Polling
prev_page: messages.html
next_page: polling.html
---

Sockets are one main component of the nanomsg API, they're used to exchanged
messages between nodes and are reprented by descriptors.  
nanomsgxx uses the [nnxx::socket](api/nnxx/socket.html) type to wrap around such descriptors and
provide automatic resource management and low level abstractions around the
C API socket routines.

Sending Messages
----------------

The [nnxx::socket](api/nnxx/socket.html) type has a *send* member function that lets us send messages
throught a socket, it comes with multiple signatures:

```c++
class socket {

  // ...

  // Working with any memory buffer.
  int send(const void *buf, size_t buflen, int flags, message_control &&ctl);
  int send(const void *buf, size_t buflen, int flags = 0);

  // Working with C-strings.
  int send(const char *str, int flags, message_control &&ctl);
  int send(const char *str, int flags = 0);

  // Working with any string-like types.
  template < typename T >
  int send(const T &str, int flags, message_control &&ctl);
  template < typename T >
  int send(const T &str, int flags = 0);

  // Zero-copy operations.
  int send(message &&msg, int flags, message_control &&ctl);
  int send(message &&msg, int flags = 0);

  // ...

};
```

As you can see the type of the first argument specifies which version will be
selected by the compiler, and other arguments are always the same:

- optional flags, that may be a combination of [nnxx::DONTWAIT](api/nnxx/namespace.html#DONTWAIT),
[nnxx::NO&#95;SIGNAL&#95;ERROR](api/nnxx/namespace.html#NO_SIGNAL_ERROR) and [nnxx::NO&#95;TIMEOUT&#95;ERROR](api/nnxx/namespace.html#NO_TIMEOUT_ERROR)
- optional [nnxx::message_control](api/nnxx/message_control.html) instance providing meta-information to the
underlying procotol

The template versions of the **send** functions accept any type that can be
iterated.

The **send** functions return the number of bytes in the sent messages, or
a negative value under some conditions if the *flags* argument isn't zero.

Receiving Messages
------------------

Receiving messages is just as easy as sending, using the **recv** member function
that exists with these signatures:

```c++
class socket {

  // ...

  // Working with any memory buffer.
  int recv(void *buf, size_t buflen, int flags, message_control &ctl);
  int recv(void *buf, size_t buflen, int flags = 0);

  // Working with any string-like types.
  template < typename T >
  T recv(int flags, message_control &ctl);
  template < typename T >
  T recv(int flags = 0);

  // Zero-copy operations.
  message recv(int flags, message_control &ctl);
  message recv(nt flags = 0);

  // ...

};
```

As you can see the type of the first argument specifies which version will be
selected by the compiler, and other arguments are always the same:

- optional flags, that may be a combination of [nnxx::DONTWAIT](api/nnxx/namespace.html#DONTWAIT),
[nnxx::NO&#95;SIGNAL&#95;ERROR](api/nnxx/namespace.html#NO_SIGNAL_ERROR) and [nnxx::NO&#95;TIMEOUT&#95;ERROR](api/nnxx/namespace.html#NO_TIMEOUT_ERROR)
- optional [nnxx::message_control](api/nnxx/message_control.html) instance where meta-information coming from
the underlying protocol will be stored.

The first version of **recv** functions return the number of bytes in the
received message, or a negative value under some conditions if the *flags*
argument isn't zero.  
The second version is templated, the template argument should be a sequence
container (string, vector, list... ) that accepts two iterators (start and end)
as argument.
The third version return an instance [nnxx::message](api/nnxx/message.html) carying a memory buffer
with a message read from the socket, the object can be evaluated to **false** if
something happened and the *flags* argument wasn't zero, for example:

```c++
nnxx::message msg { s.recv(nnxx::DONTWAIT) };

if (!msg) {
  // Check errno, it's probably be set to EAGAIN because
  // no message could be read from the socket.
  // ...
}

```

Socket Options
--------------

Options can be set on sockets, some options apply to the socket properties,
some other apply to the underlying protocol. [nnxx::socket](api/nnxx/socket.html) provides member
functions that wrap around the [nn_setsockopt](http://nanomsg.org/v0.3/nn_setsockopt.3.html)
and [nn_getsockopt](http://nanomsg.org/v0.3/nn_getsockopt.3.html) routine:

```c++
class socket {

  // ...

  void setopt(int level, int optname, const void *optval, size_t optlen);

  template < typename T >
  void setopt(int level, int optname, const T &optval);

  void getopt(int level, int optname, void *optval, size_t *optlen) const;

  template < typename T >
  void getopt(int level, int optname, T &optval) const;

  template < typename T >
  T getopt(int level, int optname) const;

  // ...

};
```

The templated version uses the other one and is a simple wrapper for convenience
so the option type must match the one required by the nanomsg C API.