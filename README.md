Hello, as I no longer have time for reverse engineering, I have decided to share some useful examples regarding the usage of Il2cppInspector C++ scaffold. In this guide, I will provide examples of how to interact with defined Il2cpp API functions.

INFO: I have created my own helper class, which you can access from the "lib" folder. I will prepare the examples based on the functions I have created.
 
# Getting the type from a class (Il2CppObject* to Type*)

```cpp
Il2CppObject* Il2CppHelper::GetTypeFromClass(const Il2CppImage* _image, const char* _namespaze, const char* _name)
{
	Il2CppClass* _targetClass = il2cpp_class_from_name(_image, _namespaze, _name);

	if (_targetClass) {
		const Il2CppType* _targetType = il2cpp_class_get_type(_targetClass);

		if (_targetType) {
			Il2CppObject* targetObject = il2cpp_type_get_object(_targetType);

			if (targetObject) {
				return targetObject;
			}
		}
	}

	return nullptr;
}
```



```cpp
const Il2CppImage* _CoreModule = _Il2CppHelper->GET_IL2CPP_IMAGE("UnityEngine.CoreModule.dll");

			if (_CoreModule) {
				Il2CppObject* _object = _Il2CppHelper->GetTypeFromClass(_CoreModule, "UnityEngine", "GameObject");

				if (_object) {
					Type* gameobjectType = reinterpret_cast<Type*>(_object);

					if (gameobjectType) {
						Object_1__Array* getAllGameObjects = Object_1_FindObjectsOfType(gameobjectType, nullptr);

						std::cout << "Gameobject count: " << getAllGameObjects->max_length << "\n";
						if (getAllGameObjects) {
							for (int i = 0; i < getAllGameObjects->max_length; i++) {
								Object_1* currentGameObject = getAllGameObjects->vector[i];

								if (GameObject_get_activeInHierarchy(reinterpret_cast<GameObject*>(currentGameObject), nullptr)) {
									std::cout << "GameObject Name: " << il2cppi_to_string(Object_1_GetName(currentGameObject, nullptr)) << "\n";
								}
								
							}
						}
					}
				}
			}
```

Output:

![Gameobjects](img/2.png)


# Getting class names and types from a specific assembly


```cpp
void Il2CppHelper::GetClassesAndNamesFromAssembly(const Il2CppImage* _image)
{
	if (_image) {
		size_t classCount = il2cpp_image_get_class_count(_image);

		std::cout << "{\n";

		for (size_t i = 0; i < classCount; ++i) {
			const Il2CppClass* _klass = il2cpp_image_get_class(_image, i);

			if (_klass) {
				char* _name = const_cast<char*>(il2cpp_class_get_name(const_cast<Il2CppClass*>(_klass)));
				char* _namespace = const_cast<char*>(il2cpp_class_get_namespace(const_cast<Il2CppClass*>(_klass)));

				std::cout << " [\n";
				std::cout << "\tName: " << _name << "\n";
				std::cout << "\tNamespace: " << _namespace << "\n";

				std::cout << " ],\n";
			}
		}

		std::cout << "\n}\n";
	}
}
```



```cpp
const Il2CppImage* _BoltDll = _Il2CppHelper->GET_IL2CPP_IMAGE("bolt.dll");

if (_BoltDll) {
    _Il2CppHelper->GetClassesAndNamesFromAssembly(_BoltDll);
}
```

or

```cpp
const Il2CppImage* _assemblyCSHARP = _Il2CppHelper->GET_IL2CPP_IMAGE("Assembly-CSharp.dll");

if (_assemblyCSHARP) {
   _Il2CppHelper->GetClassesAndNamesFromAssembly(_assemblyCSHARP);
}
```

Outputs:

![Gameobjects](img/3.png)
![Gameobjects](img/4.png)


## Getting information about any method


```cpp
void Il2CppHelper::GetMethodInfo(const Il2CppImage* _image, const char* _funcName, int argLength, const char* _class_name, const char* _class_namespace)
{
	Il2CppClass* _class = il2cpp_class_from_name(_image, _class_namespace, _class_name);

	if (_class == nullptr) return;

	const MethodInfo* methodInfo = il2cpp_class_get_method_from_name(_class, _funcName, argLength);

	if (methodInfo == nullptr) return;

	Il2CppReflectionMethod* reflectionMethod = il2cpp_method_get_object(methodInfo, _class);

	// Check if the reflectionMethod is not null
	if (reflectionMethod == nullptr) return;

	std::cout << "{\n";

	// Get the method's name from the reflectionMethod object
	const char* methodName = il2cpp_method_get_name(methodInfo);
	std::cout << "\tMethod Name: " << methodName << std::endl;

	const Il2CppType* returnType = il2cpp_method_get_return_type(methodInfo);
	std::cout << "\tReturn Type: " << il2cpp_type_get_name(returnType) << std::endl;

	// Get the parameter count of the method using il2cpp_method_get_param_count
	int parameterCount = il2cpp_method_get_param_count(methodInfo);
	std::cout << "\tParameter Count: " << parameterCount << std::endl;

	std::cout << "\t[\n";
	// Get the parameter types of the method
	for (int i = 0; i < parameterCount; i++) {
		// Get the parameter type at index i using il2cpp_method_get_param
		const Il2CppType* parameterType = il2cpp_method_get_param(methodInfo, i);

		// Get the type name of the parameter type using il2cpp_type_get_name
		const char* parameterTypeName = il2cpp_type_get_name(parameterType);

		// Print the parameter type name to the console
		std::cout << "\t\tParameter " << i << " Type: " << parameterTypeName << std::endl;
	}
	std::cout << "\t]\n";

	std::cout << "}\n";
}
```


```cpp
const Il2CppImage* _AssemblyCSharp = _Il2CppHelper->GET_IL2CPP_IMAGE("Assembly-CSharp.dll");
			
_Il2CppHelper->GetMethodInfo(_AssemblyCSharp, "SetFOV", 1, "NolanBehaviour", "");
```

Output:

![Gameobjects](img/5.png)


# Get the assemblies

```cpp
// Get the active domain
			const Il2CppDomain* domain = il2cpp_domain_get();

			// Define variables to hold the assembly list
			const Il2CppAssembly** assemblies;
			size_t size;

			// Use the il2cpp_domain_get_assemblies function to retrieve all assemblies
			assemblies = il2cpp_domain_get_assemblies(domain, &size);

			for (size_t i = 0; i < size; ++i) {
				const Il2CppAssembly* assembly = assemblies[i];

				if (assembly) {
					// Get the assembly name using il2cpp_image_get_name function
					const char* assemblyName = il2cpp_image_get_name(assembly->image);
					std::cout << assemblyName << "\n";
				}
			}
```

Output:

![Gameobjects](img/1.png)

I will continue to contribute as much as I can. For now, bye!