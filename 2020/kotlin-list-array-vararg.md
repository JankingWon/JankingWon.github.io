## 区别
首先得分清楚它们的对应关系，kotlin中没有传统的数组（`int[]`），一切皆是对象

所以`kotlin`中只有`List`，`Array`和`varargs`(可变长参数)三种
## 转换

### List转Array
```kotlin
    fun a(){
        val list : List<String> = listOf("hello","world")
        //转换1
        b(arrayOf(list))
        //转换2
        b(list.toTypedArray())
    }

    fun b(array: Array<*>){
        //Array参数函数
    }
```
### Array转List
```kotlin
    fun a(){
        val array : Array<String> = arrayOf("hello","world")
        //转换1
        b(listOf(array))
        //转换2
        b(array.toList() as List<*>)
    }

    fun b(list: List<*>){
        //List参数函数
    }
```

### Array转varargs(可变长参数)
```kotlin
    fun a(){
        val array : Array<String> = arrayOf("hello","world")
        //转换
        b(*array)
    }

    fun b(vararg strings: String){
        //可变长参数函数
    }
```
这里用到的`*`叫做传播符号(`spread operator`)，可以把`Array`转换成多个元素

### varargs(可变长参数)转Array
```kotlin
    fun a(vararg strings: String){
    	//无需转换
        b(strings)
    }

    fun b(array: Array<*>){
        //Array参数函数
    }
```

### List转varargs(可变长参数)

```kotlin
    fun a(){
        val list : List<String> = listOf("hello","world")
        //转换
        b(*list.toTypedArray())
    }
    
    fun b(vararg strings: String){
        //可变长参数函数
    }
```
类似上面的，只是先要转换成`Array`，再用传播符转成`varargs`

### varargs(可变长参数)转List
```kotlin
    fun a(vararg strings: String){
   		//转换
        b(strings.toList())
    }

    fun b(list: List<*>){
        //List参数函数
    }
```