# 走进 TypeScript

> [何镇豪](https://github.com/ElvisUpUp) / 2021-2-8

`TypeScript` 是 `JavaScript` 的强类型版本，然后它会在编译器把类型、特有语法等等编译成 `JavaScript` 。由于最终运行的是 `JavaScript` ，所以 `TypeScript` 并不会依赖于浏览器的支持，也不会带来兼容性的问题。

## 类型系统的好处

**1. 静态类型检查**

静态类型检查可以避免很多不必要的错误, 可以在编译期久发现错误，不用在运行时才发现问题。

**2. IDE 智能提示**

在没有使用 TypeScript , JavaScript 的智能提示基本完全依赖 IDE 提供的猜测。局限性就是，这种猜测可能并不正确，并且也缺乏更多的辅助信息，所以要正确使用一个类库，得不断地在文档和 IDE 之间切换，影响效率。而 TypeScript 不仅自己写的 JavaScript 有丰富的类型信息，也可以对其他纯 JS 项目进行类型标注 ([DefinitelyTyped](https://link.zhihu.com/?target=https%3A//github.com/borisyankov/DefinitelyTyped)), 便于使用者直接在 IDE 中浏览 API，效率大增。

**3. 可读性**

对于阅读代码的人来讲，各种便利的类型一目了然，更容易明白作者的意图。

## 核心

**1. 编译**

TS 的编译流程包含三步：**解析**->**转换**->**生成**

- Scanner: 从源码生成token
- Parser: 从token生成AST
- Binder: 从AST生成symbol
- Checker: 类型检查
- Emitter: 生成JS



<img :src="$withBase('/ts-compile.jpg')" alt="TypeScript">

**例子：transformer plugin**

很多UI库都支持模块的按需引用，比如：

```js
import { Button } from 'antd'
```

通过 Plugin 转换之后会被转换成：

```js
import Button from 'antd/lib/button'
```

在一个没有使用全部组件的项目里，这样做可以明显减少打包后的代码体积。

下面是一个简单的 transformer plugin 例子：

通过 [AST explorer](https://link.zhihu.com/?target=http%3A//astexplorer.net/) 可以发现，需要完成上面的转换，只需要进行两处转换，即替换 ImportClause 的子节点和替换 StringLiteral 为原来的值加上上面的 Identifier。

**转换前：**

<img :src="$withBase('/ast-before.png')" alt="TypeScript">

**转换后：**

<img :src="$withBase('/ast-after.png')" alt="TypeScript">

**代码：**

```TypeScript
import * as ts from 'TypeScript'
import * as kind from 'ts-is-kind'

function transformer(ctx: ts.TransformationContext): ts.Transformer<ts.SourceFile> {
  const visitor: ts.Visitor = (node: ts.Node): ts.Node => {
    if (kind.isSourceFile(node)) {
      return ts.visitEachChild(node, visitor, ctx)
    }

    if (kind.isImportDeclaration(node)) {
      return updateImportNode(node, ctx)
    }
    return node
  }
  return (sf: ts.SourceFile) => {
    return ts.visitNode(sf, visitor)
  }
}

function updateImportNode(node: ts.Node, ctx: ts.TransformationContext): ts.Node {
  let identifierName: string

  const visitor: ts.Visitor = node => {
    if (kind.isNamedImports(node)) {
      identifierName = node.getChildAt(1).getText()
      return ts.createIdentifier(identifierName)
    }

    if (kind.isStringLiteral(node)) {
      const libName = node.getText().replace(/[\"\']/g, '')
      if (identifierName) {
        const fileName = camel2Dash(identifierName)
        return ts.createLiteral(`${libName}/lib/${fileName}`)
      }
    }

    if (node.getChildCount()) {
      return ts.visitEachChild(node, visitor, ctx)
    }
    return node
  }

  return ts.visitEachChild(node, visitor, ctx)
}

function camel2Dash(_str: string) {
  const str = _str[0].toLowerCase() + _str.substr(1)
  return str.replace(/([A-Z])/g, ($1) => `-${$1.toLowerCase()}`)
}

function compile(sourceCode: string) {

  const source = ts.createSourceFile(
    '',
    sourceCode,
    ts.ScriptTarget.ES2016,
    true
  )

  const result = ts.transform(source, [transformer])

  const transformedSourceFile = result.transformed[0]
  const printer = ts.createPrinter()
  const resultCode = printer.printFile(transformedSourceFile)

  return resultCode
}

// import Button from "antd/lib/button";
console.log(compile('import { Button } from "antd"'))
```

其实大概思路就是：遍历节点，找到importDeclaration的节点，然后替换更新节点。

**2. 类型系统**

- **结构类型**

  TypeScript 的类型系统是基于结构子类型的，这种类型系统是一种只使用成员来描述类型的方式。它与名义类型正好相反，名义类型是通过明确的声明或类型的名称来决定的。

- **泛型**

  - 目的是为了创建可复用的组件，使其可以支持多数据类型。
  - 本质就是给类型传参

  ``````TypeScript
  interface Generic<T> {
    (x: T, y: T): T;
  }
  
  let add: Generic<number> = function add (x, y) {
    return x + y
  }
  
  add(1, 12)
  ``````

  当我们使用 `Generic<number>` 时，`<number>` 就像是参数1和12一样，方法内的类型也同样可以使用这个类型参数。

  - 可以用 extends 来对泛型约束

    ```TypeScript
    // 泛型约束
    interface Length {
      length: number
    }
    
    function loggingLength<T extends Length> (arg: T): number {
      return arg.length
    }
    ```

- **Type vs Interface**

  1. 不能通过类型别名定义联合类型来实现类
  2. 接口不能继承用联合类型定义的类型别名
  3. 类型别名不能声明合并

- **类型收缩**

  - 类型断言

    ```markdown
    <类型>值
    值 as 类型
    ```

    如果需要多处使用断言：

    ```js
    function padLeft(value: string, padding: string | number) {
        console.log((padding as number) + 3);
        console.log((padding as number) + 2);
        console.log((padding as number) + 5);
        return Array((padding as number) + 1).join(' ') + value;
    }
    ```

    可以使用类型保护

  - 类型保护

    - typeof: 用于判断 "number"，"string"，"boolean"或 "symbol" 四种类型
    - instanceof : 用于判断一个实例是否属于某个类
    - in: 用于判断一个属性/方法是否属于某个对象
    - 字面量类型保护
    - 自定义类型保护

- **Never、void、any、unknown**

  - Never是TypeScript的一种`bottom type`，即是表示永不存在的值的类型。

    ```TypeScript
    // 不相交类型的inteserction结果为never
    type IntersectionNeverType = 1 & 2
    // 是任何类型的subtype
    type Check<T> = never extends T ? true : false
    type SubTypeCheck = Check<string>
    // 布尔运算
    type result = string | never
    type result1 = string & never
    ```

  - void 当定义函数返回值类型时，表示不返回任何值；定义变量类型时，该变量只能赋值为 undefined/null

  - unknown 表示未知类型，当无法立刻确定类型时，建议先使用 unknown，到使用时再使用类型断言确定类型

    ```TypeScript
    let a: any = 1
    const b: unknown = 2;
    
    console.log(a.name) // 不会报类型错误
    console.log(b.name) // 会报类型错误
    console.log((b as {name: string}).name)
    ```

- **内置工具集**

  ##### **keyof**

  ```TypeScript
  interface Person {
    name: string;
    age: number;
  }
  
  type K1 = keyof Person; // "name" | "age"
  type K2 = keyof Person[]; // "length" | "toString" | "pop" | "push" | "concat" | "join" 
  type K3 = keyof { [x: string]: Person };  // string | number
  
  ```

  该操作符可以用于获取某种类型的所有键，其返回类型是联合类型。

  ##### **Partial**

  作用就是将某个类型里的属性全部变为可选项 `?`

  ```TypeScript
  /**
   * node_modules/TypeScript/lib/lib.es5.d.ts
   * Make all properties in T optional
   */
  type Partial<T> = {
    [P in keyof T]?: T[P];
  };
  
  ```

  更多内置工具集可以自行查阅官方文档。

- **装饰器**

  - **什么是装饰器？**

    装饰器是一种特殊的声明，可附加在类、方法、访问器、属性、参数声明上。它可以在不修改代码自身的前提下，给已有代码增加额外的行为。

  - **例子**

  ```js
  function f() {
      console.log("f(): evaluated");
      return function (target, propertyKey: string, descriptor: PropertyDescriptor) {
          console.log("f(): called");
      }
  }
  
  function g() {
      console.log("g(): evaluated");
      return function (target, propertyKey: string, descriptor: PropertyDescriptor) {
          console.log("g(): called");
      }
  }
  
  class C {
      @f()
      @g()
      method() {}
  }
  ```

  结果：

  ```js
  f(): evaluated
  g(): evaluated
  g(): called
  f(): called
  ```

  结论：

  1. 由上至下依次对装饰器表达式求值。
  2. 求值的结果会被当作函数，由下至上依次调用。

  - **类装饰器**

    - **类的语法糖**

      ```js
      class Dog {
          say() {
              console.log("汪~");
          }
      }
      ```

      等同于

      ```js
      function Dog() {}
      Object.defineProperty(Dog.prototype, "say", {
          value: function() { console.log("汪~"); },
          enumerable: false,
          configurable: true,
          writable: true
      });
      ```

    - 类装饰器

      ```js
      function isAnimal(target) {
          target.isAnimal = true;
        	return target;
      }
      
      @isAnimal
      class Dog {
          ...
      }
      
      console.log(Dog.isAnimal);    // true
      
      ```

  - **装饰器求值**

    1. *参数装饰器*，然后依次是*方法装饰器*，*访问符装饰器*，或*属性装饰器*应用到每个实例成员。
    2. *参数装饰器*应用到构造函数。
    3. *类装饰器*应用到类。

  - **@Component**

    大家其实都在 vue2.0 里面用过 TypeScript，都知道通常都会使用 vue-class-component 来配合使用。

    而`@Component`是最常用的装饰器，通过它的源码可以发现它的核心方法是`componentFactory`。

    它做了三件事：

    - 生成 options，通过一些判断来给 options 添加属性

      ```TypeScript
      const proto = Component.prototype
        Object.getOwnPropertyNames(proto).forEach(function (key) {
          if (key === 'constructor') {
            return
          }
      
          // hooks
          if ($internalHooks.indexOf(key) > -1) {
            options[key] = proto[key]
            return
          }
          const descriptor = Object.getOwnPropertyDescriptor(proto, key)!
          if (descriptor.value !== void 0) {
            // methods
            if (typeof descriptor.value === 'function') {
              (options.methods || (options.methods = {}))[key] = descriptor.value
            } else {
              // TypeScript decorated data
              (options.mixins || (options.mixins = [])).push({
                data (this: Vue) {
                  return { [key]: descriptor.value }
                }
              })
            }
          } else if (descriptor.get || descriptor.set) {
            // computed properties
            (options.computed || (options.computed = {}))[key] = {
              get: descriptor.get,
              set: descriptor.set
            }
          }
        })
      ```

    - 遍历component里的`__decorator__`属性，并执行它的factory方法

      ```TypeScript
      // decorate options
        const decorators = (Component as DecoratedClass).__decorators__
        if (decorators) {
          decorators.forEach(fn => fn(options))
          delete (Component as DecoratedClass).__decorators__
        }
      ```

    - 调用`Vue.extend(options)`创建一个组件构造器并返回

      ```TypeScript
      const superProto = Object.getPrototypeOf(Component.prototype)
      const Super = superProto instanceof Vue
      ? superProto.constructor as VueClass<Vue>
            : Vue
      const Extended = Super.extend(options)
      ...
      return Extended
      ```

#### 开发插件

1. **JSON to TS**

   可以快速完成 JSON 数据到 TS 接口的转换工作。

2. **Document This**

   可以快速对函数进行注释。

3. **Quokka.js**

   Quokka 可以在 VS Code 运行 JS/TS 代码，并显示结果。

#### 总结

这篇文章只是简单梳理了 TypeScript 的一些重要概念，对于某些概念，还需要自身去官方文档深入研究，以及在实际开发中运用来加深理解。

个人觉得，无论大型还是小型项目，TypeScript 都是必要的。