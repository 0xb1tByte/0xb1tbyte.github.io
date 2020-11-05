---
layout: post
title:  "Effective C++"
category: C++
---

Effective C++ 3/e

## 目錄

- 01~12 Accustoming Yourself to C++
- 13~17 Resource Management
- 18~25 Designs and Declarations
- 26~31 Implementations
- 32~40 Inheritance and Object-Oriented Design
- 41~48 Templates and Generic Programming
- 49~52 Customizing new and delete
- 53~55 Miscellany

## 55 Ways

### 1. View C++ as a federation of language

- C
- Object-Oriented C++
- Template C++
- STL

### 2. Prefer consts, enums and inlines to #defines

- 注意 constant pointer
- Class 專屬常數 : static const member
- enum hack
- macros 少用, 改用 inline function

### 3. Use const whenever possible

- const function & pass by reference to const
- const member function
- bitwise : 不更改物件內任何 bit
- logical : 在使用者感覺不出的情況下可更改 bit (可使用 `mutable` ) -> 通常遵守這個規則
- 當有等價實作的 member function 時, 應以 non-const member function 呼叫 const member function

  e.g. 等價實作的 operator[]

  ```cpp
  // 官方範例
  class TextBlock
  {
  public:
  const char& operator[] (std::size_t position) const
  {
      ...
      return text[position];
  }

  char& operator[](std::size_t postition)
  {
      return const_cast<char&>(static_cast<const TextBlock&>(*this)[position]);  ...(1)
  }
  }  
  ```

  - (1) : 先用 static_cast<**const** TextBlock&>(*this) 將 TextBlock& 轉為 const TextBlock&, 這樣才會呼叫 const operator[], 再將 const operator[] 回傳的結果用 **const_cast**<char&>(...) 移除 const, 回傳 non-const char&

  - 以 const member function 呼叫 non-const member function違反 logical const !

### 4. Make sure that objects are initialized before they're used

- member initialization list
- static object & local-static object -> 注意 non-local static object 的初始化時機
- Singleton : return static reference

### 5. Know what functions C++ silently writes and calls

- default constructor
- copy constructor : copy 每一個 bit
- copy assignment operator
- destructor

### 6. Explicitly disallow the use of compiler-generated functios you do not want

- private
- extend private "Uncopyable" self-define class
  - private copy constructor
  - private copy assignment operator
- function() = delete; (書中沒提到但C++11可用)

### 7. Declare destructors virtual in polymorphic base class

- 避免使用 base class pointer 去 delete derived class
  - 可能會造成局部銷毀的物件
  - 解法 : 給 base class 一個 virtual destructor
- 意圖作為 base class (含有 virtual member function) 則給一個 virtual destructor
- 解構順序 : derived class -> base class

### 8. Prevent exception from leaving destructors

- 在 destructor 中 try-catch
- 若要讓用戶處理 destructor 中可能出現異常的程式, 提供 function 給用戶在 destructor 被執行前呼叫, 並回傳執行結果給用戶自己解決問題, destructor 一樣要 try-catch

### 9. Never call virtual functions during construction or destruction

- 順序 : Base constructor -> Derived constructor

  > 當 Derived class instance 建立時會先呼叫 Base class 的 virtual function 而造成問題

- **不要**在 Base constructor 呼叫有 call virtual function 的 non-virtual 自訂義 Init() function

  e.g.

  ```cpp
  // 官方範例
  class Transaction
  {
  public:
    Transaction() { Init(); } ...(1)
    virtual void LogTransaction() const = 0;

  private:
    void Init()
    {
      ...
      LogTransaction(); ...(2)
    }
  }
  ```

  - (1) : 在 constructor 中呼叫 Init()
  - (2) : Init() 中竟然呼叫 virtual function, 不行這樣 !

- 用 constructor( **parameter** ) 往 Base constructor 傳

### 10. Have assignment operators return a reference to *this

```cpp
T& operator=(...) { return *this; }
```

### 11. Handle assignment to self in operator=

- 方法一

  暫存原 pointer 指向的值 -> new 新的並且 assign -> delete 舊的 -> return *this

  e.g.

  ```cpp
  // 官方範例
  Widget& Widget::operator=(const Widget& rhs)
  {
    Bitmap* pOrig = pb;
    pb = new Bitmap(*rhs.pb);
    delete pOrig;
    return *this;
  }
  ```

- 方法二

  在 operator= 中使用 copy and swap

### 12. Copy all parts of an object

- 記得複製所有 member

  e.g. Derived class
  - copy constructor 要在 member initialization list 呼叫 Base class 的 copy constructor
  - operator= 要呼叫 Base class 的 operator=

### 13. Use objects to manage resources

- RAII : Resource Acquisition is Initialization
- 在 constructor 取得 resource, 在 destructor 釋放 resource
- smart pointer
  - auto_ptr : copy auto_ptr 後, 原本的 auto_ptr 會指向 null
  - shared_ptr : copy shared_ptr 後, 一起指向同一個 object

### 14. Think carefully about copying behavior in resource-managing classes

- 禁止複製
- 參用計數法 (reference-count) : 沒有 reference 的時候不是刪除所指物件, 而是執行想處裡的事 (e.g. shared_ptr -> deleter)
- 複製底部資源 (deep copying)
- 轉移底部資源擁有者

### 15. Provide access to raw resources in resource-managing classes

- 顯示轉換 (explicit conversion) : get()
- 隱式轉換 (implicit conversion) : operator()

### 16. Use the same form in corresponding uses of new and delete

e.g.

- string* x = new string() -> **delete** x
- string* y = new string[100] -> **delete[]** y

### 17. Store newed objects in smart pointers in standalone statements

  要使用儲存 new object 的 smart pointer 時, 先獨立一行來處理 -> 再拿smart pointer 來使用或當作 parameter 傳入 function

e.g.

```cpp
// 官方範例
std::tr1::shared_prt<Widget> pw (new Widget);
processWidget(pw, priority());
```

### 18. Make interfaces easy to use correctly and hard to use incorrectly

- 正確使用
  - 介面一致性
  - 行為相容內建型別
- 阻止誤用
  - 建立新型別
  - 限制型別上的操作
  - 束縛物件值
  - 消除客戶的資源管理責任 (smart porinter)

### 19. **(!)** Treat class design as type design
  
設計class的注意事項

- construct, destruct
- constructor, assignment
- copy
- valid value
- inheritance graph
- type conversion
- member function
- private, protected, public
- undeclared interface
- restrict standard member function
- normalize
- 使用 derived class or non-member function or template 也可達到目的就不須定義新的class

### 20. Prefer pass-by-reference-to-const to pass-by-value

- 高效且可避免 slicing problem (由於 polymerphism, pass derived class by value 後會呼叫 base class 的 copy constructor 而失去 derived class 的特化性質, 被視為 base class object)

  e.g.

  ```cpp
  // 官方範例
  void printNameAndDisplay(Window w) ...(2)
  {
    std::cout << w.name();
    w.display();  ...(3)
  }

  WindowWithScrollBars wwsb;  ...(1)
  printNameAndDisplay(wwsb);
  ```

  - (1) : Derived class
  - (2) : pass by value -> 呼叫 Base class 的 copy constructor
  - (3) : 呼叫 Window::display 而不是 WindowWithScrollBars::display

- 不適用內建型別, STL 的 iterator 和 function object

### 21. Don't try to return a reference when you must return an object

- return by value
- **Don't** return reference or pointer to
  - local stack obejct
  - heap-allocated object
  - local static object

### 22. Declare data members private

- member -> private
- protected 不比 public 更具封裝性
- 改變時造成的程式碼破壞量與封裝性成**反比**

### 23. Prefer non-member non-friend functions to member functions

- packaging flexibility
- non-member non-friend function 可存取到 class 的資料更少
  -> 封裝性更高
- 以相同 namespace 不同 header 分離 class 與 non-member non-friend function -> 降低編相依, 方便擴展與組織 function

### 24. Declare non-member functions when type conversions should apply to all parameters

  e.g.

```cpp
class Rational
{
  ...
  const Rational operator*(const Retional& rhs) const;    ...(1)
}

->

const Rational operator*(const Rational& lhs, const Rational& rhs)  ...(2)
{
  return Rational(lhs.numerator() * rhs.numerator*(),
                  lhs.denominator() * rhs.denominator());
}
```

- (1) : 應不包含 operator\*, 即 class 內禁止 operator\*
- (2) : non-member function

### 25. Consider support for a non-throwing swap

- 如果 std::swap 效率不足於自定義的 class 或 class template, 例如使用 pimpl(pointer to implememtation), 則實作特化 swap
  - 提供一個 public swap member function, 且絕不拋出異常
  - 在與 class 或 template 相同的 namespace 下提供一個 non-member swap 並呼叫上述的 swap member function
  - 若是 class, 為 class 特化 std::swap, 並令它呼叫 swap member function
  - **(!)** 使用 swap 時, 先宣告 unsing std::swap, 而後不加 namespace 直接呼叫 swap (見底下"使用"範例)

- 範例
  - Class using pimpl

    ```cpp
    // 官方範例
    namespace WidgetStuff
    {
    class Widget
    {
    public:
    ...
    void swap(Widget& other)
    {
      using std::swap;    ...(1)
      swap(pImpl, other.pImpl);   ...(2)
    }
    ...
    }

    void swap(Widget& a, Widget& b)
    {
      a.swap(b);
    }
    }   // namepsace WidgetStuff

    namespace std
    {
    template<>  ...(3)
    void swap<Widget>(Widget& a, Widget& b) ...(4)
    {
      a.swap(b);
    }
    }   // namespace std
    ```

    - (1) : 必要, 讓 compiler 找不到特化的 swap 時使用 default std::swap
    - (2) : 自定義 swap 的實作 -> 只需替換 pImpl 指標就好
    - (3) : 全特化 total template specialization
    - (4) : "針對 T 是 Widget" 的特化版本, 以防仍不小心使用 std::swap 則會使用此特化版本

  - Class template

    ```cpp
    // 官方範例
    namespace WidgetStuff
    {
    template<typename T>
    class Widget { ... };
    ...
    template<typename T>
    void swap(Widget<T>& a, Widget<T>& b)
    {
      a.swap(b);
    }
    }   // namespace WidgetStuff

    // 以下不合法
    namespace std
    {
      template<typename T>
      void swap(Widgt<T>& a, Widget<T> b)
      {
        a.swap(b);
      }
    }   // namespace std
    ```

    - C++ 只允許對 class templates 偏特化, function templates 不行
    - 可以全特化 std 內的 templates, 但不能在 std 裡面加東西 (templates or classes ot functions), 在此為添加了一個 overloading function templates
    - 結論 : 當為 class template 時, swap 不需要在 std 裡處理任何東西

- 使用

  ```cpp
  template<typename T>
  void doSomething(T& obj1, T& obj2)
  {
      using std::swap;    ...(1)
      ...
      swap(obj1, obj2);   ...(2)
      ...
  }
  ```

  - name lookup rules (argument-dependent lookup)

      swap function under global or namespace -> std specialization swap -> std::swap

  - (1) : 讓 std::swap 在 function 內可見
  - (2) : 不加 namespace 修飾, 讓 compliler 決定最佳解

### 26. Postpone variable definitions as long as possible

- 真要用到了才宣告
- 更好是使用 copy constructor
- 比較

e.g.[A] : variable definition **before** loop

```cpp
Widget w;
for(...)
{
  w = ...;
}
```

- 1 constructor
- 1 destructor
- n assign

e.g.[B] : variable definition **in** loop

```cpp
for(...)
{
  Widget w(...);
}
```

- n constructor
- n destructor

結論 => `if( assign 成本 < constructor + destructor || performance_sensitive ) ? [A] : [B]`

### 27. Minimize casting

- e.g

  ```cpp
  Derived d;
  Base* pb = &d;
  ```

  - 有時候並不相同
  - 在執行期 Derived* 上會有 offset 以取得正確的 Base* pointer

- e.g. Call base class function

  ```cpp
  class Derived : public Base
  {
  public:
    virtual void func()
    {
      // 以下錯誤示範
      static_cast<Base>(*this).func();    ...(1)
      ...
    }
  }
  ```

  - (1) : 喚醒的是 static_case\<Base\>(*this) 的**暫時副本**的 func(), 即在"當前物件之 base class 成分"的的副本上呼叫 Base::func(), 如果有改動物件內容時, 改動的是副本而不是當前物件

    => (1) 應改為 `Base::func()`

- e.g. Base pointer 要執行 Derived class 的 function
  - 少用 dynamic_cast
  - 用 Derived*
  - 在 Base class 多加 virtual function

- 把轉型隱藏在 function 裡, e.g. toString()

### 28. Avoid returning "handles" to object internals

- 少 return reference to private member
- 有可能造成 dangling handles

### 29. Strive for exception-safe code

- Exception-safe function
  - 基本承諾 : 若出異常, 資料或物件仍有效, 但執行結果可能不合預期
  - 強烈保證 : 成功, 或回到原狀態
  - 不拋擲 (nothrow) 保證

### 30. Understand the ins and outs of inlining

- 過度 inline
  - 程式碼膨脹
  - 額外 pagin
  - 降低 instruction cache hit rate
- 80-20 法則 : 一個程式80%執行時間花在20%的程式碼上

### 31. **(!)** Minimize compilation dependencies between files

- pimpl idiom (pointer to implementation)
- 以宣告的依存性取代定義的依存性
- Header file 盡可能自我滿足 (full and declaration-only)
  - object-ref or object-ptr first
  - class declaration first (<-> class definition)
      => 將"提供class定義式"(#include)的義務從Header轉移到"內含函式呼叫"的客戶檔
  - 為宣告式和定義式提供不同的 Header
    - header.h
    - headerfwd.h   <- include 這個取代 forward declaration
- 方法
  - Handle Classes : pimpl
  - Interface Classes

### 32. Make sure public inheritance models "is-a"

### 33. Avoid hiding inherited names

- derived class 內的名稱會掩蓋 base class 的名稱 => 不要取同樣的名稱, **就算 function signature 不一樣也會被掩蓋**
- 使用 base class function 的方法
  - using
  - forwarding functions : 在 function 中呼叫 base function

### 34. Differentiate between inheritance of interface and inheritance of implementation

```text
function       | 介面繼承 | 預設實作繼承 | 強制實作繼承
---------------|---------|-------------|------------
pure virtual   |    V    |             |
impure vertual |    V    |      V      |
non-virtual    |    V    |             |      V
```

### 35. Consider alternatives to virtual functions

- Template Method Pattern
  - non-virtual interface (NVI)

    讓 non-virtual 的 public member function 呼叫 private virtual member function, 而 derived class 可以重新定義 virtual member function 的實作

- Strategy pattern
  - function pointer
  - \<functional\>
  - 古典 strategy : object

### 36. Never redefine an inherited non-virtual function

- bound
  - non-virtual function : statically bound
  - virtual function : dynamically bound
- call
  - non-virtual

      假設 derived class 與 base class 定義相同名稱的 member function, 以 Base* 及 Derived* 指向相同 derived 物件並呼叫相同的 member function => Base* 會 call base function, derived* 會 call derived function

  - virtual

      皆為呼叫 Derived class 的 function

### 37. Never redefine a function's inherited default parameter value

- 使用 Base class 的 pointer 指向 dynamic type 時, 當 function 有 default 值會皆為 Base class 的
- default parameter 為 static binding
- 使用 temaplate pattern

### 38. Model "has-a" or "is-implemented-in-terms-of" through composition

- composition
  - application domain : "has-a"
  - implementation domain : "is-implemented-in-terms-of"

### 39. Use private inheritance judiciously

- private inheritance : "implemented-in-terms-of"
- private inheritance 的替代方案
  - 巢狀 private class, 並以 public 繼承
  - 在 class 外定義 public 繼承, 再 composite
- private 繼承使用時機
  - derived class 要存取 protected base class member
  - 重新定義繼承而來的 virtual function
  - EBO ( Empty base optimization)

### 40. Use multiple inheritance judiciously

- virtual 繼承會增加大小, 速度, 初始化(及賦值)的複雜度 => 所以盡量是 virtual base class 不帶任何資料(member)
- 使用時機 : class 同時 public 繼承 interface + private 繼承協助實作的 class

### 41. Understand implicit interfaces and complile-time polymorphism

```text
         | interface          | polymorphism
---------|--------------------|--------------------------------------
class    | explicit           | virtual function
         | (signature)        | (runtime)
---------|--------------------|--------------------------------------
template | implicit           | 具現化 + function overload resolution
         | (valid expression) | (compile time)

```

- class 和 template 的 interface 皆在 compile time 檢查
- template implicit interface
  - operator overloading
  - 可被 implicit 轉換而呼叫物件的 operator

  => 表面上要 return 的 function 或 operator 不一定要成立, 隱式轉換後成立即可 !

### 42. Understand the two meanings of typename

- template 宣告參數型別, typename 等於 class
- template 內出現的名稱
  - dependent names : template 中相依 template 參數的名稱
  - nested dependent name
  - nested dependent type name
  - non-dependent names
- 注意

  e.g.

  ```cpp
  // 官方範例
  template<typename C>
  void print2nd(const C& container)
  {
    C::const_iterator* x;
    ...
  }
  ```

  假設 C::const_iterator 不是型別, 如 static_member 或 global variable, 則程式執行結果會出乎意料 ! (template 遇到 nested dependent name 預設不把它當成型別)

  - 在 template 中以 `typename C::const_iterator` 宣告
  - 不能用 typename 修飾的例外
    - base class lists
    - member initialization list

### 43. Know how to access names in templatized base classes

- template<> : total template sprcialization
- 使用 templatized base class 的 member function
  - this->func()
  - using base_class::func()
  - base_class::func()

### 44. Factor parameter-independent code out of templates

- commonality and variability analysis
- 編譯期最佳化 vs. 執行檔大小
- template 程式碼不該與某個造成膨脹的 template 參數產生相依關係
- template 中造成程式碼膨脹的解決方法
  - non-type template parameter : 以 function parameter 或 class member 取代 template 參數
  - type template parameter : 以相同 binary representation的 instantiation type 共享實作碼 (e.g. void*)

### 45. Use member function templates to accept "all compatible types"

- 同一個 template 的不同具現體 (instantiations) 之間並不存在固有關係
- 作法
  - generic copy constructor : 以 member initialization list 在編譯期檢查隱式轉換合法性
  - generic assignment operator

  e.g.

  ```cpp
  // 官方範例
  template<typename T>
  class SmartPtr
  {
  public:
    template<typename U>
    SmartPtr(const SmartPtr<U>& other)
        : heldPtr(other.get()) { ... }
    ...
    T* get() const { return heldPtr; }
    ...
  private:
    T* heldPtr;
  }
  ```

- member template 不改變語言規則, 所以有 generic copy constructor (generic assignment operator) 還是需要宣告一般的 copy constructor (assigment operatpr)

### 46. Define non-member functions inside templates when type conversions are desired

- 參考 Rule24 的例子
- 若為隱式轉換, function template 無法推導回原本的 type
  - 解法

    將函式定義為 class template 內部的 friend 函式

    (Class template 不依賴 template 引數推導)

    e.g.

    ```cpp
    // 官方範例
    template<typename T>
    class Rational
    {
    public:
      ...
      friend const Rational operator* (const Rational& lhs,
                                        const Tational& rhs) ...(1)
      {
        return doMultiply(lhs, rhs);
      }
    };

    template<typename T>
    const Rational<T> doMultiply (const Rational<T>& lhs,
                                  const Rational<T>& rhs)    ...(2)
    {
      return Rational(lhs.numerator() * rhs.numerator(),
                      lhs.denominator() * rhs.denominator());
    }

    ```

    - (1) : **friend** member function
    - (2) : helper -> non-member template function

- 在一個 class template 內, template 名稱可被用來作為 "template 和其參數" 的簡寫 (e.g. Rational 即為   Rational\<T\>)
- 結論 : 當 class template 有與此 template 相關且與 template 的參數有隱式型別轉換, 將函式定義為 class template 內部的 friend 函式

### 47. Use traits class for information about types

- Traits 不是 C++ 關鍵字, 是一種技術, 用來取得 type 相關的資訊
- 可用於 build-in type 和 user-defined type
- 型別的 traits 資訊必須位於型別本身之外 -> 放在 template 及一或多個特化版

  e.g. 以 iterator 為例

  ```cpp
  template<typename IterT>
  struct iterator_traits  ...(1)
  {
    typedef typename IterT::iterator_category iterator_category;  ...(2)
    ...
  }

  template<typename IterT>
  struct iterator_traits<IterT*>  ...(3)
  {
    typedef random_access_iterator_tag iterator_category;
    ...
  }
  ```

  - (1) : traits 習慣以 struct 實作, 但常稱為 traits class
  - (2) : 以 iterator_category 代表 IterT::iterator_category
  - (3) : partial specialization, 因 IterT* 行為即為 random_access_iterator_tag
  - 使用範例

    ```cpp
    // STL list iterator
    template< ... >
    class list
    {
    public:
      class iterator
      {
      public:
        typedef bidirectional_iterator_tag iterator_category;
      }
    }

    // get information
    iterator_traits<list<int>::iterator_category>::iterator_category tag;
    ```

    - `iterator_traits<list<int>::iterator_category>::iterator_category tag;` 等價於 `bidirectional_iterator_tag tag;`
    - 分析上述(2)並帶入list\<int\>範例

      ```cpp
      template<typename list<int>::iterator_category>
      struct iterator_traits
      {
        typedef typename list<int>::iterator_category iterator_category;  ...(4)
        ...
      }
      ```

      - (4) : 即以 iterator_category 代表 list\<int\>::iterator_category 的 typedef (而 list\<int\>::iterator_category 的 typedef 為bidirectional_iterator_tag) => typedef 的 typedef

- 為 build-in type 定義偏特化 template
- 若 template 有 if...else... 判斷 "type" (執行期), 可用 overloading 在編譯期判斷確定

### 48. Be aware of template metaprogramming

- 編譯期程式 -> 早期偵錯 & 高效
- 範例

  ```cpp
  template<unsigned n>
  struct Factorial
  {
    enum { value = n * Factorial<n-1>::value };
  };

  template<>
  struct Factorial<0>
  {
    enum { value = 1 };
  }
  ```

- 使用時機
  - 確保度量正確
  - 優化矩陣運算
  - 生成客製 design pattern 實作

### 49. Understand the behavior of the new-handler

- operator new 無法滿足記憶體配置需求時
  - 拋出異常
  - return null pointer (舊 compiler)
- Header : \<new\> => std::set_new_handler
- 當 operator new 無法滿足記憶體申請時會不斷呼叫 new_handler 函式直到足夠的 memory
  - 配置更多記憶體在呼叫後給它用
  - 安裝另一個 new_handler 接續處理
  - 卸除 new_handler, null 給 `set_new_handler`
  - 拋出 bad_alloc
  - 不返回, abort or exit
- 可實作各 class 的專屬 new_handler => static std::new_handler member
- 可用 template class 實作 set_new_handler(std::new_handler) 與 operator new => class 使用 CRTP(curiously recurring template pattern) 繼承之

### 50. Understand when it makes sense to replace new and delete

- 何時要自定義 new and delete operator ?
  - 檢測運用上的錯誤 : memory leaks, overruns, underruns
  - 強化效能 : 內建的為一般性, fragmentation
  - 收集使用上的數據統計

### 51. Adhere to convention when writing new and delete

- 編寫 new 和 delete 的規則
- operator new
  - 回傳正確的值
  - 記憶體不足呼叫 new_handling function
  - 零記憶體需求
  - 避免掩蓋正常形式的 new, 見 Rule52
- C++ 規定 0 bytes request (operator new) 也得回傳一個合法的指標
- operator new 裡面有個 while 直到記憶體成功配置 return 指向已配置記憶體的 pointer, 否則配置失敗呼叫 new handling function 執行 Rule49 相關處理或直接return
- C++ free standing object 必為非0大小
- derived class : operator new 的 size 需處裡
  - Base 的 new 裡要判斷 => `(size != sizeof(Base))`
  - `return ::operator new(size)` <= 呼叫標準 operator new

### 52. Write placement delete if you write placement new

- e.g. `Widget* pw = new Widget;`
  - 先 operator new 再 Widget default constructor
  - 假設 new 成功但 constructor 失敗, 必須歸還記憶體, 否則會造成 memory leaks. 若 constructor 拋出異常, C++ 執行期處理 => 呼叫 operator new 對應的 operator delete
- 自定義 placement new(delete)
  - (O) 唯一額外引數是 void*
  - 任意額外引數
- placement new 要有對應的 placement delete
- 不要遮蓋標準 new, 見 Rule33, 且使用 `::operator new (delete)`

### 53. Pay attention to compliler warnings

### 54. Familiarize youself with the std library, include TR1

### 55. Familiarize yourself with Boost
