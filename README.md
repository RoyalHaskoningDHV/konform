# Portable validations for Kotlin
Note: this project was forked because its original maintainer seems to have abandoned it. All credit for contributions goes to their authors. We would like to thank [Niklas Lochschmidt](https://twitter.com/niklas_l) for starting this project and contributing almost all work so far.


  - **✅ Type-safe DSL**
  - **🔗 Multi-platform support** (JVM, JS)
  - **🐥 Zero dependencies**

### Installation

Add the konform bintray repository to your build.gradle

```groovy
repositories {
    maven { url "https://dl.bintray.com/rhdhv/konform" }
}
```

Depending on your type of Kotlin project add one of these dependencies:

- JVM:   
`implementation 'com.rhdhv:konform:0.1.1'`
- JS:  
`implementation 'com.rhdhv:konform-js:0.1.1'`
- Common:  
`implementation 'com.rhdhv:konform-common:0.1.1'`

### Use

Suppose you have a data class like this:

```Kotlin
data class UserProfile(
    val fullName: String,
    val age: Int?
)
```

Using the Konform type-safe DSL you can quickly write up a validation

```Kotlin
val validateUser = Validation<UserProfile> {
    UserProfile::fullName {
        minLength(2)
        maxLength(100)
    }

    UserProfile::age ifPresent {
        minimum(0)
        maximum(150)
    }
}
```

and apply it to your data

```Kotlin
val invalidUser = UserProfile("A", -1)
val validationResult = validateUser(invalidUser)
```

since the validation fails the `validationResult` will be of type `Invalid` and you can get a list of validation errors by indexed access:

```Kotlin
validationResult[UserProfile::fullName]
// yields listOf("must be at least 2 characters")

validationResult[UserProfile::age]
// yields listOf("must be equal or greater than 0")
```

In case the validation went through successfully you get a result of type `Valid` with the validated value in the `value` field.

```Kotlin
val validUser = UserProfile("Alice", 25)
val validationResult = validateUser(validUser)
// yields Valid(UserProfile("Alice", 25))
```

### Advanced use

You can define validations for nested classes and use them for new validations

```Kotlin
val ageCheck = Validation<UserProfile> {
    UserProfile::age required {
        minimum(18)
    }
}

val validateUser = Validation<UserProfile> {
    UserProfile::fullName {
        minLength(2)
        maxLength(100)
    }
    
    run(ageCheck)
}
```

It is also possible to validate nested data classes and properties that are collections (List, Map, etc...)

```Kotlin
data class Person(val name: String, val email: String?, val age: Int)

data class Event(
    val organizer: Person,
    val attendees: List<Person>,
    val ticketPrices: Map<String, Double?>
)

val validateEvent = Validation<Event> {
    Event::organizer {
        // even though the email is nullable you can force it to be set in the validation
        Person::email required {
            pattern(".+@bigcorp.com") hint "Organizers must have a BigCorp email address"
        }
    }

    // validation on the attendees list
    Event::attendees {
        maxItems(100)
    }

    // validation on individual attendees
    Event::attendees onEach {
        Person::name {
            minLength(2)
        }
        Person::age {
            minimum(18) hint "Attendees must be 18 years or older"
        }
        // Email is optional but if it is set it must be valid
        Person::email ifPresent {
            pattern(".+@.+\..+") hint "Please provide a valid email address (optional)"
        }
    }

    // validation on the ticketPrices Map as a whole
    Event::ticketPrices {
        minItems(1) hint "Provide at least one ticket price"
    }

    // validations for the individual entries
    Event::ticketPrices onEach {
        // Tickets may be free in which case they are null
        Entry<String, Double?>::value ifPresent {
            minimum(0.01)
        }
    }
}
```

Errors in the `ValidationResult` can be accessed again using the index access method. In case of `Iterables` and `Arrays` you use the numerical index and in case of `Maps` you use the key as string.

```Kotlin
// get the error messages for the first attendees age if any
result[Event::attendees, 0, Person::age]

// get the error messages for the free ticket if any
result[Event::ticketPrices, "free"]
```

##### Maintainer
[Kilian Grashoff](https://github.com/kiliangrashoff/) - Digital Lab - Royal HaskoningDHV

##### Original Author

[Niklas Lochschmidt](https://twitter.com/niklas_l)

##### License

[MIT License](https://github.com/konform-kt/konform/blob/master/LICENSE)
