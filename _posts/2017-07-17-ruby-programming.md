---
layout: default
title:  "Ruby祖先链和幽灵方法"
date:   2017-07-17 12:00:00 +0800
categories: ruby update
---
# {{ page.title }}

## 一、祖先类
在Ruby的世界里，“祖先链”是一个很重要的概念。祖先链为我们展示了一个完整的类继承路径，同时它也是Ruby中方法查找的路径

### 1.Ruby继承
  ruby的继承是单继承，可通过`:superclass`方法查看类的父类，在定义类的时候用 `<` 进行类继承，可以使用`:superclass`查看父类：
  ```ruby
  class A;end

  class B < A
  end
  B.superclass  # => A
  ```
  动态定义类时，通过传入参数的方式实现继承：
  ```ruby
  class A;end

  B = Class.new(A){}
  B.superclass  # => A
  ```
### 2.Ruby中的Mixin
  虽然ruby是单继承，但是我们可以通过Mixin机制实现类似多继承的效果
  ```ruby
  class A
    def a
      puts 'this is method a'
    end
  end

  module M_B
    def b
      puts 'this is method a'
    end
  end

  module M_C
    def c
      puts 'this is method c'
    end
  end

  class B < A
    include M_B
    include M_C
  end

  B.instance_methods   # => [:a, :b, :c, ...]
  ```
  > include 是将模块中的实例方法引入到类中作为实例方法，extend 则是作为类方法。

  > 当模块被 include 的时候，会触发模块的 included 方法

### 3.include和prepend的区别
  `include`和`prepend`都是将模块中的实例方法引入到类中作为实例方法，但它们之间也有些不同，使用`include`时，模块在祖先链中的位置是在类的后面，而`prepend`则是将模块放在类的前面：
  ```ruby
  class A;end
  module B;end
  module C;end

  class D < A
    include B
    prepend C
  end
  C.ancestors   # => [C, D, B, A, Object, Kernel, BasicObject]
  ```

  > 当你想要通过引入模块来覆盖某个类的方法时，应使用`prepend`。

  > 引入多个模块时，模块在祖先链中的位置时根据引入顺序决定的。

## 二、幽灵方法: method_missing

  `:method_missing`，又称幽灵方法，当调用不存在的方法时，就会触发`:method_missing`的调用
  
  ```ruby
  class A
    def method_missing(method_name, *args)
      puts "missing #{method_name}"
    end
  end

  A.new.a   # => 'missing :a'
  ```
  与`:send`配合使用：

  ```ruby
  class A
    def find(arg)
      puts arg.inspect
    end

    def method_missing(method_name, *args)
      reg = /^find_by_([a-zA-Z]+)/
      super unless res = reg.match(method_name.to_s)
      send :find, res[1] => args.first
    end
  end

  a = A.new
  a.find_by_name 'winse'  # => {'name'=>'winse'}
  a.a                     # => NoMethodError
  ```

  与`:define_method`配合使用来动态定义方法：
  
  ```ruby
  class A
    def find(arg)
      puts "find #{arg.inspect}"
    end

    def method_missing(method_name, *args)
      reg = /^find_by_([a-zA-Z]+)/
      super unless res = reg.match(method_name.to_s)
      self.class.send :define_method, method_name do |arg|
        find res[1] => arg
      end
      send method_name
    end
  end

  a = A.new
  a.find_by_name 'winse'  # => 'find {"name"=>"winse"}'
  a.a                     # => NoMethodError
  ```
  方法查找

  ```ruby
  class A
    def method_missing(n, *args)
      puts "A -> #{n}"
      super
    end

    def self.method_missing(n, *args)
      puts "A --> #{n}"
      super
    end
  end

  module C
    def method_missing(n, *args)
      puts "C -> #{n}"
      super
    end

    def self.method_missing(n, *args)
      puts "C --> #{n}"
      super
    end
  end

  module D
    def method_missing(n, *args)
      puts "D -> #{n}"
      super
    end

    def self.method_missing(n, *args)
      puts "D --> #{n}"
      super
    end
  end

  class B < A
    include C
    prepend D

    def method_missing(n, *args)
      puts "B -> #{n}"
      super
    end

    def self.method_missing(n, *args)
      puts "B --> #{n}"
      super
    end
  end

  B.a   # => 'B --> a'
        # => 'A --> a'
        # => 'NoMethodError...'

  B.new.a # => 'D -> a'
          # => 'B -> a'
          # => 'C -> a'
          # => 'A -> a'
          # => 'NoMethodError...'
  ```
  从上面的代码我们可以看出：调用不存在的类方法时，ruby会从祖先链中从下往上进行方法查找，但不会调用模块中的method_missing，而调用不存在的实例方法时，ruby会从祖先链中一层一层往上找，不会对模块和类进行区分。

  > 使用`:method_missing`时，最好留出一条分支调用父类的`:method_missing`，即`super`

## 参考资料
  [Ruby 的方法查找与 method_missing](https://ruby-china.org/topics/21304)