# propagate_const

A smart-pointer wrapper for maintaining const-correctness of a smart-pointer when a member of a class.

Maintains const-correctness in pimpl pattern and dependancy injection patterns as
might be used in TDD;
whereby member pointers in a class are an abstraction of logical members of the class

example 1:
```
class MyClassImpl
{
public:
    MyClassImpl() {}
    void function() { std::cout << "non-const function called" << std::endl; } // non-const function
    void function() const { std::cout << "const function called" << std::endl; } // const function 
};

class MyClass
{
public:
    MyClass() : impl_(new MyClassImpl()) {}
    void function() { impl_->function(); }
    void function() const { impl_->function(); } // both const and non-const functions use same call to pointer
private:
    propagate_const<std::unique_ptr<MyClassImpl>> impl_; // pointer to class implementation
};

int main()
{
    MyClass myClassInstance; // create class instance
    myClassInstance.function(); // calls non-const function, outputs: 'non-const function called'

    const auto& myConstClassInstance = myClassInstance; // create constant reference to class
    myConstClassInstance.function(); // calls const function, outputs: 'const function called'

    exit(EXIT_SUCCESS);
}
```

example 2:

```
class MyCompositePartClass
{
public:
    MyCompositePartClass() {}
    void function() { std::cout << "non-const function called" << std::endl; }
    void function() const { std::cout << "const function called" << std::endl; }
};

class MyClassFactory
{
public:
    MyClassFactory() {}
    virtual std::shared_ptr<MyCompositePartClass> CreateMyCompositePartClass()
    {
        return std::make_shared<MyCompositePartClass>();
    }
    virtual ~MyClassFactory() {}
};

class MyClass
{
public:
    MyClass(std::shared_ptr<MyClassFactory> classFactory = std::make_shared<MyClassFactory>())
    {
        composite_ = classFactory->CreateMyCompositePartClass();
    }
    void function() { return composite_->function(); }
    void function() const { return composite_->function(); }
private:
    propagate_const<std::shared_ptr<MyCompositePartClass>> composite_;
};

int main()
{
    MyClass myClassInstance;
    myClassInstance.function(); // calls non-const function, outputs: 'non-const function called'

    const auto& myConstClassInstance = myClassInstance;
    myConstClassInstance.function(); // calls const function, outputs: 'const function called'

    exit(EXIT_SUCCESS)
}
```

output:
```
non-const function called
const function called
```
