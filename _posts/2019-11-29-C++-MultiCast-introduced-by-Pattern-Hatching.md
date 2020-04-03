---
layout: post
title: "C++ MultiCast introduced by Pattern Hatching"
author: "Jonghyun Ho"
categories: Design Pattern
tags: [Design Pattern]
image: pattern_hatching.jpg
---

Pattern Hatching 이라는 책에 소개된 C++ 로 구현된 MultiCast 예제이다.

Pattern Hatching 은 디자인 패턴이라는 개념을 정의했던 Gang of Four(GoF) 4명의 저자 중 한 명인
존 블리시데스(John Vlissides)가 실제 사례에 대해 패턴이 어떻게 적용될 수 있는지를 소개한 책이다.

MultiCast는 데이터를 전송할 때 하나의 노드가 여러 노드에 동시에 전송할 수 있는 개념이다.

저자는 다음과 같은 상황에 MultiCast 를 쓸 것을 권장하고 있다.
```
- 특정 객체의 클래스가 다른 객체로부터 정보를 받는 데 관심이 있을 수 있다.
- 정보는 임의의 구조와 복잡도를 가지며 소프트웨어가 발전함에 따라 달라질 수 있다.
- 정보 교환은 정적으로 타입 안전성이 좋아야 한다.
```

소개된 MultiCast 예제는 이벤트 기반으로 동작하고,
확장 가능하고 타입 안전성이 좋아 유용하게 사용할 수 있을 것 같아 메모해 두려고 한다.

``` c++
#include <algorithm>
#include <iostream>
#include <list>
#include <string>

using namespace std;

template <class T>
class TEvent {
 public:
  class Handler {
   public:
    Handler() { TEvent<T>::Register(this); }
    virtual ~Handler() { TEvent<T>::Unregister(this); }
    virtual int HandleEvent(const T& t) = 0;
  };

  typedef std::list<Handler*> HandlerList;

  static void Register(Handler* handler) {
    registry_.push_back(handler);
  }

  static void Unregister(Handler* handler) {
    typename std::list<Handler*>::iterator it;

    for (it = registry_.begin(); it != registry_.end(); it++) {
      if (*it == handler) {
        registry_.remove(handler);
        return;
      }
    }
  }

  static void Notify(TEvent<T>* t) {
    typename std::list<Handler*>::iterator it;

    for (it = registry_.begin(); it != registry_.end(); it++) {
      (*it)->HandleEvent(*(T*)t);
    }
  }

  void Notify() { T::Notify(this); }

 private:
  static HandlerList registry_;
};

class CoinInsertedEvent : public TEvent<CoinInsertedEvent> {};
class CoinReleaseEvent : public TEvent<CoinReleaseEvent> {};
class ProductDispensedEvent : public TEvent<ProductDispensedEvent> {};

class CoinChanger : public CoinReleaseEvent::Handler,
                    public ProductDispensedEvent::Handler {
 public:
  CoinChanger(string name) : name_(name) {}

 public:
  int HandleEvent(const ProductDispensedEvent& event) {
    cout << name_ << " : Product dispensed." << endl;
    return 0;
  }

  int HandleEvent(const CoinReleaseEvent& event) {
    cout << name_ << " : Coin released." << endl;
    return 0;
  }

 private:
  string name_;
};

TEvent<CoinInsertedEvent>::HandlerList
       TEvent<CoinInsertedEvent>::registry_;
TEvent<CoinReleaseEvent>::HandlerList
       TEvent<CoinReleaseEvent>::registry_;
TEvent<ProductDispensedEvent>::HandlerList
       TEvent<ProductDispensedEvent>::registry_;

int main() {
  CoinChanger coinChanger("coin changer 1");
  CoinReleaseEvent coinReleaseEvent;
  ProductDispensedEvent productDispensedEvent;

  coinReleaseEvent.Notify();
  productDispensedEvent.Notify();

  cout << endl;

  CoinChanger coinChanger2("coin changer 2");

  CoinReleaseEvent coinReleaseEvent2;
  coinReleaseEvent2.Notify();

  return 0;
}
```
