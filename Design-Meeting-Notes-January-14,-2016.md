UWP, EF, and .NET Native
========================

RC2 will bring improved support for EF on Universal Windows Platform.

`Scaffold-Directives` powershell command
----------------------------------------
Most .NET Native errors can be resolved by adding the right [runtime directive (rdxml)](https://msdn.microsoft.com/en-us/library/dn600639(v=vs.110).aspx) to the project. To simplify, we are introducing a tool that will automate rdxml generation. The new powershell command `Scaffold-Directives` generates (most of) the rdxml necessary for .NET Native.

How it generates rdxml
----------------------
The command compiles the user's project and loads all DbContext types in the project. [`RuntimeTypeDiscoverer`](https://github.com/aspnet/EntityFramework/blob/3b693c7ee789f117c3e1a7f5f2cde8aeb298393c/src/EntityFramework.Commands/Design/Internal/RuntimeTypeDiscoverer.cs) loads the model and executes logic to determine what kinds of generic types may exist at runtime.

One step in this logic is scanning all loaded assemblies for the attribute [`[CallsMakeGenericMethod()]`](https://github.com/aspnet/EntityFramework/blob/3b693c7ee789f117c3e1a7f5f2cde8aeb298393c/src/EntityFramework.Core/Internal/CallsMakeGenericMethodAttribute.cs). This attribute should be declared on all methods that make use of `MakeGenericMethod`. Based on the arguments in the attribute and the user's model, `RuntimeTypeDiscoverer` will attempt to construct the generic method and include the appropriate rdxml for this type.

`CallsMakeGenericMethod` accepts placeholder types that can be used to represent user types. See [`TypeArgumentCategory`](https://github.com/aspnet/EntityFramework/blob/9aab7992d1825ebc903ca321d420dcb5cabb2e65/src/EntityFramework.Core/Internal/TypeArgumentCategory.cs)

Example: from `PropertiesAccessorFactory`, [See source here](https://github.com/aspnet/EntityFramework/blob/9aab7992d1825ebc903ca321d420dcb5cabb2e65/src/EntityFramework.Core/Metadata/Internal/PropertyAccessorsFactory.cs#L18-L41)
```c#
        [CallsMakeGenericMethod(nameof(CreateGeneric), typeof(TypeArgumentCategory.Properties))]
        [CallsMakeGenericMethod(nameof(CreateGeneric), typeof(HashSet<object>))]
        public virtual PropertyAccessors Create([NotNull] IPropertyBase propertyBase)
            => (PropertyAccessors)typeof(PropertyAccessorsFactory).GetTypeInfo().GetDeclaredMethod(nameof(CreateGeneric))
                .MakeGenericMethod((propertyBase as IProperty)?.ClrType
                                   ?? (((INavigation)propertyBase).IsCollection()
                                       ? typeof(HashSet<object>)
                                       : typeof(object)))
                .Invoke(null, new object[] { propertyBase });

        private static PropertyAccessors CreateGeneric<TProperty>(IPropertyBase propertyBase)
        {
            // ...
        }
```

Contributors beware: unsafe API calls
-------------------------------------
Our code makes use of APIs that are unsafe to use on .NET Native without runtime directives. The following calls should be avoided if possible. Where used, `RuntimeTypeDiscoverer` may need to be altered to account for possible types that can be constructed at runtime.

- `System.Reflection.MethodInfo.MakeGenericMethod()`
- `System.Type.MakeGenericType()`
- `System.Linq.Expressions.Expression.Lambda()`