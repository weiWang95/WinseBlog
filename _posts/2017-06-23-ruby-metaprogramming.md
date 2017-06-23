---
layout: default
title:  "Ruby元编程"
date:   2017-06-23 12:00:00 +0800
categories: ruby update
---
# {{ page.title }}
## 一、打开类
  打开类，又称猴子补丁(monkey patch)，就是重新对类进行方法的新增、覆盖等，例：

  ```ruby
  class A
    def self.func_a;end
  end

  A.a              # => nil
  A.methods false  # => [:func_a]

  class A
    def self.func_a
      'this is func a'
    end

    def self.func_b;end
  end

  A.a              # => 'this is func a'
  A.methods false  # => [:func_a, :func_b]
  ```
  > 这是一种很不安全的操作，因为你能打开所有的类，如：Object, Kernel等，假如不小心覆盖了ruby内核方法，系统有可能直接崩溃。


## 二、define_method
### 1. 作用描述
  `:define_method`作用为定义方法，例：
  ```ruby
  define_method :method_name do |arg1, arg2|
    puts arg1
  end

  method_name 'world', ''  # => world
  ```
### 2. 使用场景
  定义大量代码结构相同的方法时可以使用，如：
  ```ruby
  [:name, :age, :sex].each do |attr|
    define_method attr do
      instance_variable_get attr
    end

    define_method "#{attr.to_s}=" do |val|
      instance_variable_set attr, val
    end
  end
  ```
  与`:method_missing`配合使用，见method_missing简绍。


## 三、method_missing
  `:method_missing`又称幽灵方法，当对象调用不存在的方法时，会触发此方法的调用，例：
  ```ruby
  class A
    def method_missing(method_name, *args, &block)
      puts "Method #{method_name.to_s}" not exist"
    end
  end

  a = A.new
  a.func_a   # => Method func_a not exist
  ```
