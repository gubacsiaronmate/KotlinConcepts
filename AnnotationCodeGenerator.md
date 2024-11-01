# Lesson: Create a Kotlin Annotation for Code Generation

In this lesson, you will learn how to create a custom annotation in Kotlin and use Kotlin Symbol Processing (KSP) to generate code based on the annotated elements.

## Step 1: Define the Annotation

First, define a custom annotation. This annotation will be used to mark the classes for which you want to generate code.

### Code

```kotlin
@Target(AnnotationTarget.CLASS)
@Retention(AnnotationRetention.SOURCE)
annotation class GenerateToString
```

### Explanation

- **`@Target(AnnotationTarget.CLASS)`**: Indicates that this annotation can only be applied to classes.
- **`@Retention(AnnotationRetention.SOURCE)`**: Specifies that this annotation is only available in the source code and is discarded by the compiler.

## Step 2: Create an Annotation Processor

We'll use Kotlin Symbol Processing (KSP) for annotation processing. KSP will allow us to generate code based on the annotated elements.

### Add Dependencies

Add the necessary KSP dependencies to your `build.gradle.kts`:

```kotlin
dependencies {
    implementation("com.google.devtools.ksp:symbol-processing-api:1.0.0")
    ksp("com.google.devtools.ksp:symbol-processing-api:1.0.0")
}
```

### Processor Code

Create an annotation processor that processes classes annotated with `@GenerateToString` and generates a `toString()` method for them.

#### ToStringProcessor.kt

```kotlin
import com.google.devtools.ksp.processing.*
import com.google.devtools.ksp.symbol.*
import com.google.devtools.ksp.validate

class ToStringProcessorProvider : SymbolProcessorProvider {
    override fun create(environment: SymbolProcessorEnvironment): SymbolProcessor {
        return ToStringProcessor(environment)
    }
}

class ToStringProcessor(
    private val environment: SymbolProcessorEnvironment
) : SymbolProcessor {

    override fun process(resolver: Resolver): List<KSAnnotated> {
        val symbols = resolver.getSymbolsWithAnnotation(GenerateToString::class.qualifiedName!!)
        val ret = symbols.filterNot { it.validate() }.toList()
        
        symbols
            .filter { it is KSClassDeclaration && it.validate() }
            .forEach { it.accept(ToStringVisitor(), Unit) }
        
        return ret
    }

    inner class ToStringVisitor : KSVisitorVoid() {
        override fun visitClassDeclaration(classDeclaration: KSClassDeclaration, data: Unit) {
            val className = classDeclaration.simpleName.asString()
            val packageName = classDeclaration.packageName.asString()
            val properties = classDeclaration.getAllProperties().map { it.simpleName.asString() }

            val fileSpec = environment.codeGenerator.createNewFile(
                Dependencies(false, classDeclaration.containingFile!!),
                packageName,
                "${className}_ToString"
            )

            fileSpec.bufferWriter().use { writer ->
                writer.write("package $packageName\n\n")
                writer.write("fun $className.toString(): String {\n")
                writer.write("    return \"${className}(")
                properties.forEachIndexed { index, property ->
                    writer.write("$property=\${this.$property}")
                    if (index < properties.size - 1) {
                        writer.write(", ")
                    }
                }
                writer.write(")\"\n")
                writer.write("}\n")
            }
        }
    }
}
```

### Explanation

- **ToStringProcessorProvider**: Provides the `ToStringProcessor` instance.
- **ToStringProcessor**: Processes symbols annotated with `@GenerateToString` and generates the `toString` method.
- **ToStringVisitor**: Visits each annotated class and generates a `toString` method using the class properties.

## Step 3: Apply the Annotation

Use the custom annotation on a class, and let the processor generate the code for you.

### Example

#### Person.kt

```kotlin
@GenerateToString
data class Person(val name: String, val age: Int)
```

### Generated Code

After running the annotation processor, a new file `Person_ToString.kt` will be generated with the following content.

```kotlin
package com.your.package.name

fun Person.toString(): String {
    return "Person(name=${this.name}, age=${this.age})"
}
```

## Putting It All Together

Ensure you have the following in your project structure:

1. **Annotation Definition**: `GenerateToString.kt`
2. **Processor Code**: `ToStringProcessorProvider.kt` and `ToStringProcessor.kt`
3. **Annotated Class**: `Person.kt`

This is a basic example illustrating how to create an annotation and use KSP to generate a `toString()` method for annotated classes in Kotlin. You can expand and customize this further based on your requirements.

---

Now you have a complete example of how to create an annotation that generates code in Kotlin using KSP.