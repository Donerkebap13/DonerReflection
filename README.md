
# DonerReflection
![Doner Reflection](https://i.imgur.com/oDC0Brl.png)

A C++14 header-only library to provide information about your class members.
## What is DonerReflection?
DonerReflection is a C++14 header-only library that provides you basic information about your class members, such as **member type** or **name**.

It also provides a way for applying changes to this members by the use of **Resolver classes**. 

An example of the usage of this system can be found in [Doner Serializer](https://github.com/Donerkebap13/DonerSerializer), a serialization library that uses the power of **DonerReflection** to serialize and deserialize your class data from/to JSON.

## Downloading

You can acquire stable releases [here](https://github.com/Donerkebap13/DonerReflection/releases).

Alternatively, you can check out the current development version with:

```
git clone https://github.com/Donerkebap13/DonerReflection.git
```
## Contact

You can contact me directly via [email](mailto:donerkebap13@gmail.com).
Also, if you have any suggestion or you find any bug, please don't hesitate to [create a new Issue](https://github.com/Donerkebap13/DonerReflection/issues).
If you decide to start using **DonerReflection** in your project, I'll be glad to hear about it and post it here in the main page as an example!
## Some interesting usages
### Serialization
As in the example mentioned above, [Doner Serializer](https://github.com/Donerkebap13/DonerSerializer) is a serialization library that uses the power of **DonerReflection** to serialize and deserialize your class data from/to JSON. This library can be easily extended to support any other kind of serialization such as XML or Binary thank to the power of **DonerReflection**.
### Editor 
By using the information provided by **DonerReflection** and with the help of a UI library such as **QT**, you could end up with something similar to this dinamically:
![Reflection UI Example](https://i.imgur.com/UHWwsfY.png)
## How to use it
DonerReflecion uses **three simple macros** to expose your class data.
```c++
namespace Foo
{
	struct Bar
	{
		int m_int;
		float m_float;
		const char* m_char;
	}
}
// IMPORTANT!!
// It is mandatory that this macro calls happen always outside of any namespace
DONER_DEFINE_REFLECTION_DATA(Foo::Bar,
	DONER_ADD_NAMED_VAR_INFO(m_int, "intFoo"),
	DONER_ADD_NAMED_VAR_INFO(m_float, "floatBar"),
	DONER_ADD_NAMED_VAR_INFO(m_float, "charMander")
)
```
As simple as that. Those macros will define a struct containing all the registered members info for that class. As the comment say, **this calls should be done outside of any namespace**. Otherwise it won't work.
Also, if you don't need a name, you can use the following macro instead:
```c++
DONER_DEFINE_REFLECTION_DATA(Foo::Bar,
	DONER_ADD_VAR_INFO(m_int),
	DONER_ADD_VAR_INFO(m_float),
	DONER_ADD_VAR_INFO(m_float)
)
```
The name info in this case will be the same as the variable name.

In order to access protected/private members, you need to use the following macro:
```c++
namespace Foo
{
	class Bar
	{
	DONER_DECLARE_OBJECT_AS_REFLECTABLE(Bar)
	private:
		int m_int;
		float m_float;
		const char* m_char;
	}
}
// IMPORTANT!!
// It is mandatory that this macro calls happen always outside of any namespace
DONER_DEFINE_REFLECTION_DATA(Foo::Bar,
	DONER_ADD_NAMED_VAR_INFO(m_int, "intFoo"),
	DONER_ADD_NAMED_VAR_INFO(m_float, "floatBar"),
	DONER_ADD_NAMED_VAR_INFO(m_float, "charMander")
)
```
By doing this you allow DonerReflection to access private members information.
### Inheritance
DonerReflection doesn't support inheritance per se. If you want to serialize class members inherited from its upper class, you need to do as follows:
```c++
class Foo
{
protected:
	int m_int;
}

class Bar : public Foo
{
DONER_DECLARE_OBJECT_AS_REFLECTABLE(Bar)
private:
	float m_float;
}

DONER_DEFINE_REFLECTION_DATA(Bar,
	DONER_ADD_VAR_INFO(m_int),
	DONER_ADD_VAR_INFO(m_float)
)
```
Even if the upper class have some reflection data defined, **this information is not transitive**, so you need to re-declare it for any children class.
## How to access Member data
All the mentioned macros end up creating a struct containing all the infor about your types inside a specialization of struct ``SDonerReflectionClassProperties<>``
```c++
class Bar
{
DONER_DECLARE_OBJECT_AS_REFLECTABLE(Bar)
private:
	float m_float;
}

DONER_DEFINE_REFLECTION_DATA(Bar,
	DONER_ADD_VAR_INFO(m_float)
)
// ...
SDonerReflectionClassProperties<Bar>::s_properties; // tuple with all the info
SDonerReflectionClassProperties<Bar>::s_propertiesCount; // number of elements in the tuple
```
Each element in the tuple is of type ``  DonerReflection::SProperty<ClassType, MemberType)>``
```c++
DonerReflection::SProperty<Bar, float> property;
property.m_name; // Registered name
property.m_member; // Pointer to the class member
// To use m_member
Bar object;
object.*(property.m_member) = 1337.f;
```
## Resolvers
**DonerReflection** supports the possibility to apply a specific action to all members of a class with the usage of a **Resolver Class**.
```c++
Bar object;
rapidjson::Document root; // Example taken from project DonerSerializer
APPLY_RESOLVER_TO_OBJECT(object, Foo::CBarResolver, root, /*>Any extra parameter needed by your resolver*/)
```
Where ``Foo::CBarResolver`` must implement a static method ``Apply`` as following:
```c++
namespace Foo
{
	class CBarResolver
	{
	public:
		template<typename MainClassType, typename MemberType>
		static void Apply(const DonerReflection::SProperty<MainClassType, MemberType>& property, MainClassType& object, rapidjson::Document& root)
		{
			AnyInternalMehtod(property.m_name, object.*(property.m_member), root);
		}
	private:
		static void AnyInternalMehtod(const char* name, std::int32_t value, rapidjson::Document& root)
		{
			root.AddMember(rapidjson::GenericStringRef<char>(name), value, root.GetAllocator());
		}
		// ...
		// Specializations for each possible supported type
	};
}
```
For a better understanding of the usage of **Resolvers**, again, I recommend you yo have a look at [Doner Serializer](https://github.com/Donerkebap13/DonerSerializer).