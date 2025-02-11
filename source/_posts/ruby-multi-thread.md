---
title: Ruby Multi Thread
date: 2025-02-11 11:12:20
tags:
    - Ruby
    - multi-thread
categories:
    - Ruby
---

# Ruby Multi-Thread
"Rails' threading model largely depends on the Ruby interpreter being used. By default, Ruby is installed with MRI (Matz's Ruby Interpreter), which employs a Global Interpreter Lock (GIL). The GIL restricts execution to one thread at a time, meaning that even when multiple threads are created in a Ruby application, only one thread can execute Ruby code at any given moment. As a result, threads in MRI are queued and executed sequentially, limiting the benefits of multi-threading within a single process."

<!-- more -->

We can use a easy code to show this:
```ruby
require 'benchmark'

Benchmark.bm do |x|
  x.report('Single Thread') do
    10_000_000.times{ 2+2 }
  end

  x.report('Multi Thread') do
    a = Thread.new{ 5_000_000.times{ 2+2 } }
    b = Thread.new{ 5_000_000.times{ 2+2 } }
    a.join
    b.join
  end
end
```
We can see from the output that the execution time of Single Thread and Multi Thread is almost the same.

|  use  | system | total | real |
|------|-------|-------|-------|
| Single Thread | 0.234827 | 0.001438 | 0.236265 (0.244640) |
| Multi Thread | 0.234102 | 0.000981 | 0.235083 (0.237760) |

![execution time](https://ithelp.ithome.com.tw/upload/images/20250211/20171817TWNHlMvCpZ.png)

## Can GIL make Multi Thread safe?
Although the GIL system ensures that only one thread executes Ruby code at a time, it does not guarantee precise control over thread switching. In real-world Ruby programs, the system does not always check the status of threads when switching between them, which can lead to race conditions.

A race condition occurs when multiple threads access and modify shared resources simultaneously, leading to inconsistent or unpredictable program behavior. Here's a simple example:

```ruby
class Sheep
  def initialize
    @shorn = false
  end

  def shorn?
    @shorn
  end

  def shear!
    puts "shearing..."
    @shorn = true
  end
end

sheep = Sheep.new

5.times.map do
  Thread.new do
    unless sheep.shorn?
      sheep.shear!
    end
  end
end.each(&:join)
```

We have a single instance of a sheep and create five threads that access the same resource (the sheep instance). With the GIL we just discussed, one would expect only one shearing to occur, right? (The reasoning is that once the first thread executes the shear! method and updates @shorn to true, all subsequent threads should skip the shearing operation.) However, the reality isn't as straightforward as we might think...

![sheep be cutted](https://ithelp.ithome.com.tw/upload/images/20250211/201718170qqpnbQ6Ee.png)

In reality, the sheep ends up being sheared 5 times.

### Race Condition
Although the GIL only ensures that one thread is running at any given moment, it does not guarantee that thread switches will occur at safe points in the code. The lack of proper synchronization allows multiple threads to simultaneously evaluate shorn? as false before any of them updates @shorn to true. As a result, all five threads proceed to execute the shear! method, leading to multiple shearings.

![ideal multi-thread switching](https://ithelp.ithome.com.tw/upload/images/20250211/20171817H3fJigWSQF.png)

In the ideal multi-thread switching process (as shown in the image), after the first thread finishes executing the shear! method, it updates the @shorn status to true. When the second thread switches in, it checks the @shorn status and correctly skips executing the shear! method because it sees that @shorn is already true.
but reality is not like this


![reality multi-thread switching](https://ithelp.ithome.com.tw/upload/images/20250211/20171817y8Q2GtqICg.png)

In reality (as shown in the image), the system might switch to the second thread before the first thread finishes execution. Since the first thread hasn't updated the @shorn status yet, the second thread proceeds to execute the shear! method again. Once the second thread finishes and the system switches back to the first thread, the first thread finally updates the @shorn status.
This race condition explains why the sheep ends up being sheared multiple times.

### Lazy initialization
A race condition caused by multi-threading can occur during lazy initialization using the ||= operator. Imagine this scenario:
```ruby
@logger ||= Logger.new
```
Is it possible that this initialization will face the same race condition issue?

---

# The real multi-thread in Ruby
Since the default interpreter in Ruby is MRI, and MRI has a built-in GIL system, we cannot achieve true multi-threading.
Is there a way to implement real multi-threading in Ruby?
Yes! By using a different Ruby interpreter.

## JRuby
MRI is implemented in C, while JRuby is based on a Java implementation. Since JRuby does not have a GIL system, we can achieve true multi-threading by using it.

### Install JRuby
Since JRuby is based on a Java implementation, you need to ensure that Java is installed before using JRuby.
```bash
java -version
```
If you haven't installed Java, you will see an error message like this:
```bash
Command 'java' not found
```
```bash
The operation couldn’t be completed. Unable to locate a Java Runtime.
Please visit http://www.java.com for information on installing Java.
```
That mean you need to install Java first.
1. Use Homebrew to install openjdk:
   ```bash
   brew install openjdk
   ```
2. Configuring environment variables
   ```bash
    sudo ln -sfn $(brew --prefix openjdk)/libexec/openjdk.jdk /Library/Java/JavaVirtualMachines/openjdk.jdk
    ```
3. Setting JAVA_HOME
   ```bash
   echo 'export JAVA_HOME="$(/usr/libexec/java_home)"' >> ~/.zshrc
   source ~/.zshrc
   ```
4. Use rvm to install and switch to jruby
   ```bash
    rvm install jruby
    rvm use jruby
    ```
---

### Run Multi-thread example in JRuby
Now, we can use the example code mentioned earlier to experience the difference between single-threaded and multi-threaded behavior.

```ruby
require 'benchmark'

Benchmark.bm do |x|
  x.report('Single Thread') do
    10_000_000.times{ 2+2 }
  end

  x.report('Multi Thread') do
    a = Thread.new{ 5_000_000.times{ 2+2 } }
    b = Thread.new{ 5_000_000.times{ 2+2 } }
    a.join
    b.join
  end
end
```

|  use  | system | total | real |
|------|-------|-------|-------|
| Single Thread | 0.239000 | 0.000000 | 0.239000 (0.237845) |
| Multi Thread | 0.116000 | 0.000000 | 0.116000 (0.115995) |

![multi-thread result](https://ithelp.ithome.com.tw/upload/images/20250211/20171817gDNxwdf29b.png)

We can clearly observe that multi-threading performs significantly faster than single-threading

---
 
# Reference
1. [Ruby 無人知曉的 GIL](https://ruby-china.org/topics/28415)