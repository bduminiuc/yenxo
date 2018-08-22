# rproject

Some *add-in*'s which enables the traits listed below for user defined types:
* serialization/deserialization in application boundaries;
* comparation;
* pushing to `std::ostream`.

Uses `boost::hana` in order to obtain reflection in C++.
Uses `rapidjson` in order to parse and serialize JSON.

## Examples

Typical use case of `mixins::Var`:

```cpp
// rproject
#include <variant.hpp>
#include <variant_mixins.hpp>

struct Hobby : mixin::Var<Hobby> {  // enables serialization/deserialization
    int id;
    std::string description;
};

struct Person : mixin::Var<Person> {
    std::string name;
    int age;
    Hobby hobby;
};

int main() {
    auto const json = R"(
        {
            "name": "Efendi",
            "age": 20,
            "hobby": {
                "id": 10,
                "description": "Barista"
            }
        }
    )";

    Person const p = Person::fromVariant(Variant::from(rapidjson::Document().Parse(json))));
}

// add reflection
BOOST_HANA_ADAPT_STRUCT(Hobby, id, description);
BOOST_HANA_ADAPT_STRUCT(Person, name, age, hobby);
```

Typical use case of `mixin::OStream`, `EqualityComparison`:

```cpp
// rapidjson
#include <variant.hpp>
#include <variant_mixins.hpp>
#include <ostream_mixins.hpp>
#include <comparison_mixins.hpp>

// std
#include <iostream>

struct Hobby
        : mixin::Var<Hobby>
        , mixin::OStream<Hobby>              // enables `std::ostream`
        , mixin::EqualityComparison<Hobby> { // enables `==` and `!=`
    int id;
    std::string description;
};

struct Person
        : mixin::Var<Person>
        , mixin::OStream<Person>
        , mixin::EqualityComparison<Person> {
    std::string name;
    int age;
    Hobby hobby;
};

int main() {
    auto const json = R"(
        {
            "name": "Efendi",
            "age": 20,
            "hobby": {
                "id": 10,
                "description": "Barista"
            }
        }
    )";

    Hobby hobby{
        10,
        "Barista"
    };

    Person expected{
        "Efendi",
        20,
        hobby
    };

    auto const person = Person::fromVariant(Variant::from(rapidjson::Document().Parse(json)));
    assert(person == expected);
}

BOOST_HANA_ADAPT_STRUCT(Hobby, id, description);
BOOST_HANA_ADAPT_STRUCT(Person, name, age, hobby);
```
