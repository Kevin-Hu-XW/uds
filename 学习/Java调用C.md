#  Java调用C++的几种方式



JNI（Java Native Interface）
JNI定义了一种公用的语法，当Java和c/c++都遵循这样的语法时就可以互相调用(也可调用汇编等其余语言)。JNI不能直接调用c/c++的库，必须使用java编写调用函数，生成C头文件，再利用C头文件编写C代码，生成动态库，最后JNI使用新生成的动态库完成执行。
过程繁琐，需增加改动Java和C/C++的程序。

JNA（Java Native Access）
JNA提供了一组Java工具类，用于在运行期间动态访问系统本地库（native library：如Window的dll）而不需要编写任何Native/JNI代码，省去了对c/c++程序的再封装。



## 一、JNI

### 原理

​	Java本身提供了一个工具包（Java动态接口，Java Native Interface,JNI)，专门用来为其他语言提供自己的规范，而Java与C++之间交流的通道，正是需要用到JNI才能实现。但这里需要注意的是JNI本身是Java对外的一份说明书，对外说明如何跟Java交流，并不能用它来约束Java本身来“妥协”其他平台（语言）的规范。

​	有了JNI这份规范之后，我们要理清楚一下Java是如何从C++那里获得方法（函数）的。首先，C++要拿到这份规范（JNI），才能跟着Java的规范来实现自己的方法（函数）。也就是说，将C++“嵌入（不知道这么说恰不恰当）”Java程序，是需要“委屈”C++跟着Java的意思走的，毕竟要进人家门做客，还得遵从主人家的意思对吧。

​	然后，Java对于C++来说还是个外乡人，Java的规范（JNI）世界公认，但是他听不懂Java的方法（函数）想要他做什么。Java想要C++帮他个忙，因此将自己的方法（.java文件）交给了翻译官Java虚拟机（Java Virtual Machine，JVM），然后翻译官翻译出了一份“二进制文件（.class）”，这份文件是Java的世界语言（机器语言），由他写出，然后几乎所有的代码平台（语言）都可以获取他的内容，但是没有办法直接读它，但是，他们都可以翻译他从而知道他的意思。同时，Java还贴心地为C++翻译了一份C++能看懂的文件（.h），于是C++便拿了这份文件，装到自己的工具包里，当做规范来使用，也就能实现Java的意图了。

​	自此，Java对C++的传递已完成。

​	接着C++在自己的工作室（.cpp）实现了Java的意图。

​	但是，C++实现了方法，Java又看不懂了，这个世界语言（.class）C++又写不出来，咋办呢？诶，这时候有个万金油出现了，他叫动态链接库（Dynamic Link Library，DLL）他的功能很强大，虽然在这里出现有些杀鸡用牛刀，但是也不失为一种高效便捷的方法。DLL用某种办法，将C++的方法存到了他的库文件（.dll）中。这个库文件比较优秀，Java本地有工具（API）可以直接在几个熟悉的地方（环境变量%JAVA_HOME%）找到他，而只要C++导出的这个库文件放到Java熟悉的几个地方，就能够供Java使用。

​	自此，Java和C++之间，关于这些方法的通道就彻底打通了。



### 步骤

1.新建Demo类，声明一个本地方法

```
class HelloWorld {
    public native void displayHelloWorld();

}
```





2.将写好的类进行编译

通过命令 **javac  HelloWorld.java **生成 .class文件

然后使用javac -h生成扩展名为.h的头文件   **javac -h ./ HelloWorld.java**

##### 注意：生成成功之后，列表出现.h文件。值得注意的是，此头文件包含了我们Java包的名字，如果没有包将不会生成带包名的命名



接下来是使用C++实现java中声明的方法

3. 新建一个dll项目，先将上面生成的头文件加入到项目中，然后将jdk目录下的inlcude文件夹下的jni.h和jawt.h以及include/win32文件夹下的jin_md.h和jawt_md.h文件添加在上面的dll项目中

4. 本地方法实现

   创建Dll_Native.h引入头文件

```C++
#include "javaCpp_HelloWorld.h"
#include <iostream>
#include "pch.h"
#include "framework.h"
JNIEXPORT void JNICALL Java_javaCpp_HelloWorld_displayHelloWorld(JNIEnv*, jobject);
```
​     创建Dll_native.cpp

​	

```C++
#include "Dll_native.h"
JNIEXPORT void JNICALL  Java_javaCpp_HelloWorld_displayHelloWorld(JNIEnv*, jobject) {
	std::cout << "I from C++" << std::endl;
}
```

5. 将生成的dll文件复制到java工程的根目录下及调用结果

```java
package javaCpp;
 
class HelloWorld {

    public native void displayHelloWorld();

    public static void main(String[] args) {
        //System.out.println(System.getProperty("java.library.path"));
        System.load("F:\\test\\src\\javaCpp\\DemoDll.dll");
        new HelloWorld().displayHelloWorld();
    }
}

I from C++
```





### 参数传递

JNI字段描述符

​	    “([Ljava/lang/String;)V” 它是一种对函数返回值和参数的编码。这种编码叫做JNI字段描述符（JavaNative Interface FieldDescriptors)。一个数组int[]，就需要表示为这样"[I"。如果多个数组double[][][]就需要表示为这样 "[[[D"。也就是说每一个方括号开始，就表示一个数组维数。多个方框后面，就是数组 的类型。

​	    如果以一个L开头的描述符，就是类描述符，它后紧跟着类的字符串，然后分号“；”结束。比如"Ljava/lang/String;"就是表示类型String；

​	"[I"就是表示int[];

​	"[Ljava/lang/Object;"就是表示Object[]。

JNI方法描述符，主要就是在括号里放置参数，在括号后面放置返回类型，如下：

（参数描述符）返回类型

当一个函数不需要返回参数类型时，就使用”V”来表示。

比如"()Ljava/lang/String;"就是表示String f();

"(ILjava/lang/Class;)J"就是表示long f(int i, Class c); 

"([B)V"就是表示void(byte[] bytes);

 

| Java 类型     | 符号                                                         |
| ------------- | ------------------------------------------------------------ |
| *Boolean*     | Z                                                            |
| *Byte*        | B                                                            |
| *Char*        | C                                                            |
| *Short*       | S                                                            |
| *Int*         | I                                                            |
| *Long*        | J                                                            |
| *Float*       | F                                                            |
| *Double*      | D                                                            |
| *Void*        | V                                                            |
| *objects*对象 | 以"L"开头，以";"结尾，中间是用"/" 隔开的包及类名。比如：Ljava/lang/String;如果是嵌套类，则用$来表示嵌套。例如 "(Ljava/lang/String;Landroid/os/FileUtils$FileStatus;)Z" |

另外[数组](https://so.csdn.net/so/search?q=%E6%95%B0%E7%BB%84&spm=1001.2101.3001.7020)类型的简写,则用"["加上如表A所示的对应类型的简写形式进行表示就可以了，

比如：[I 表示 int [];[L/java/lang/Object;表示Objects[],另外。引用类型（除基本类型的数组外）的标示最后都有个";"

例如：

"()V" 就表示void Func();

"(II)V" 表示 void Func(int, int);

"(Ljava/lang/String;Ljava/lang/String;)I".表示 int Func(String,String)

**注：JNI字段描述符没有包装类型，所以在Java方法中的类型声明应该使用基础类型**



#### 一、在Navtive层返回字符串

java层原型方法为

```java
public class HelloJni {  
    ...  
    public native String getAJNIString();  
    ...  
} 
```

 Native层该方法实现为 :

```c++
//返回字符串  
JNIEXPORT jstring JNICALL Java_com_xxx_getAJNIString(JNIEnv * env, jobject obj)  
{  
    jstring str = env->newStringUTF("HelloJNI");  //直接使用该JNI构造一个jstring对象返回  
    return str ;  
}  
```



#### 二、在Native层返回一个int型的二维数组

Java层原型方法

```java
public class HelloJni {  
    ...  
    //参数代表几行几列数组 ，形式如：int a[dimon][dimon]  
    private native int[][] getTwoArray(int dimon) ;   
    ...  
}    
```

Native层该方法实现为 :

```c++
//通过构造一个数组的数组， 返回 一个二维数组的形式  
JNIEXPORT jobjectArray JNICALL Java_com_xxx_getTwoArray  
  (JNIEnv * env, jobject object, jint dimion)  
{  
      
    jclass intArrayClass = env->FindClass("[I"); //获得一维数组 的类引用，即jintArray类型  
    //构造一个指向jintArray类一维数组的对象数组，该对象数组初始大小为dimion  
    jobjectArray obejctIntArray  =  env->NewObjectArray(dimion ,intArrayClass , NULL);  
  
    //构建dimion个一维数组，并且将其引用赋值给obejctIntArray对象数组  
    for( int i = 0 ; i< dimion  ; i++ )  
    {  
        //构建jint型一维数组  
        jintArray intArray = env->NewIntArray(dimion);  
  
        jint temp[10]  ;  //初始化一个容器，假设 dimion  < 10 ;  
        for( int j = 0 ; j < dimion ; j++)  
        {  
            temp[j] = i + j  ; //赋值  
        }  
          
        //设置jit型一维数组的值  
        env->SetIntArrayRegion(intArray, 0 , dimion ,temp);  
        //给object对象数组赋值，即保持对jint一维数组的引用  
        env->SetObjectArrayElement(obejctIntArray , i ,intArray);  
  
        env->DeleteLocalRef(intArray);  //删除局部引用  
    }  
  
    return   obejctIntArray; //返回该对象数组  
}  
```



#### 三、在Native层操作Java层的类 ：读取/设置类属性

Java层原型方法：

```java
public class HelloJni {  
    ...  
    //在Native层读取/设置属性值  
    public native void native_set_name() ;  
    ...  
      
    private String name = "I am at Java" ; //类属性  
}     
```

Native层该方法实现为 :

```c++
//在Native层操作Java对象，读取/设置属性等  
JNIEXPORT void JNICALL Java_xxx_HelloJni_native_1set_1name  
  (JNIEnv *env , jobject  obj )  //obj代表执行此JNI操作的类实例引用  
{  
   //获得jfieldID 以及 该字段的初始值  
   jfieldID  nameFieldId ;  
  
   jclass cls = env->GetObjectClass(obj);  //获得Java层该对象实例的类引用，即HelloJNI类引用  
  
   nameFieldId = env->GetFieldID(cls , "name" , "Ljava/lang/String;"); //获得属性句柄  
  
   if(nameFieldId == NULL)  
   {  
       cout << " 没有得到name 的句柄Id \n;" ;  
   }  
   jstring javaNameStr = (jstring)env->GetObjectField(obj ,nameFieldId);  // 获得该属性的值  
   const char * c_javaName = env->GetStringUTFChars(javaNameStr , NULL);  //转换为 char *类型  
   string str_name = c_javaName ;    
   cout << "the name from java is " << str_name << endl ; //输出显示  
   env->ReleaseStringUTFChars(javaNameStr , c_javaName);  //释放局部引用  
  
   //构造一个jString对象  
   char * c_ptr_name = "I come from Native" ;  
     
   jstring cName = env->NewStringUTF(c_ptr_name); //构造一个jstring对象  
  
   env->SetObjectField(obj , nameFieldId , cName); // 设置该字段的值  
}  
```



#### 四、在Native层操作Java层的类：回调Java方法 

Java层原型方法：

```java
public class HelloJni {  
    ...  
    //Native层回调的方法实现  
    public void callback(String fromNative){       
        System.out.println(" I was invoked by native method  ############# " + fromNative);  
    };  
    public native void doCallBack(); //Native层会调用callback()方法  
    ...   
      
    // main函数  
    public static void main(String[] args)   
    {  
        new HelloJni().ddoCallBack();  
    }     
} 
```

 Native层该方法实现为 :

```c++
//Native层回调Java类方法  
JNIEXPORT void JNICALL Java_com_xxx_HelloJni_doCallBack  
  (JNIEnv * env , jobject obj)  
{  
     //回调Java中的方法  
  
    jclass cls = env->GetObjectClass(obj);//获得Java类实例  
    jmethodID callbackID = env->GetMethodID(cls , "callback" , "(Ljava/lang/String;)V") ;//或得该回调方法句柄  
  
    if(callbackID == NULL)  
    {  
         cout << "getMethodId is failed \n" << endl ;  
    }  
    
    jstring native_desc = env->NewStringUTF(" I am Native");  
  
    env->CallVoidMethod(obj , callbackID , native_desc); //回调该方法，并且传递参数值  
} 
```



#### 五、复杂对象

**Student.java**类

```java
package com.uds.saas.demo;

public class Student {
    
    private int age;
    private String name;

    public Student() {
    }

    public Student(int age, String name) {
        this.age = age;
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Student{" +
                "age=" + age +
                ", name='" + name + '\'' +
                '}';
    }
}

```





##### 在Native层返回一个复杂对象

Java层的方法对应为：

```java
public class HelloJni {  
    ...  
    //在Native层返回一个Student对象  
    public native Student nativeGetStudentInfo() ;  
    ...   
}
```

  Native层该方法实现为 :

```c++
//返回一个复杂对象  
JNIEXPORT jobject JNICALL Java_com_xxx_HelloJni_nativeGetStudentInfo  
  (JNIEnv * env, jobject obl)  
{  
    //关于包描述符，这儿可以是 com/xxx/jni/Student 或者是 Lcom/xxx/jni/Student;   
    //   这两种类型 都可以获得class引用  
    jclass stucls = env->FindClass("com/xxx/jni/Student"); //或得Student类引用  
  
    //获得得该类型的构造函数  函数名为 <init> 返回类型必须为 void 即 V  
    jmethodID constrocMID = env->GetMethodID(stucls,"<init>","(ILjava/lang/String;)V");  
  
    jstring str = env->NewStringUTF(" come from Native ");  
  	//构造一个对象，调用该类的构造函数，并且传递参数  
    jobject stu_ojb = env->NewObject(stucls,constrocMID,11,str);  
    return stu_ojb ;  
}
```





##### 从Java层传递复杂对象至Native层

Java层的方法对应为：

```java
public class HelloJni {  
    ...  
    //在Native层打印Student的信息  
    public native void  printStuInfoAtNative(Student stu);  
    ...   
}  
```

Native层该方法实现为 :  

```c++
//在Native层输出Student的信息  
JNIEXPORT void JNICALL Java_com_xxx_HelloJni_printStuInfoAtNative  
  (JNIEnv * env, jobject obj,  jobject obj_stu) //第二个类实例引用代表Student类，即我们传递下来的对象  
{  
      
    jclass stu_cls = env->GetObjectClass(obj_stu); //或得Student类引用  
  
    if(stu_cls == NULL)  
    {  
        cout << "GetObjectClass failed \n" ;  
    }  
    //下面这些函数操作，我们都见过的。O(∩_∩)O~  
    jfieldID ageFieldID = env->GetFieldID(stucls,"age","I"); //获得得Student类的属性id   
    jfieldID nameFieldID = env->GetFieldID(stucls,"name","Ljava/lang/String;"); // 获得属性name 
  
    jint age = env->GetIntField(objstu , ageFieldID);  //获得属性值  
    jstring name = (jstring)env->GetObjectField(objstu , nameFieldID);//获得属性值  
  
    const char * c_name = env->GetStringUTFChars(name ,NULL);//转换成 char *  
   
    string str_name = c_name ;   
    env->ReleaseStringUTFChars(name,c_name); //释放引用  
      
    cout << " at Native age is :" << age << " # name is " << str_name << endl ;   
}  
```



##### 在Native层返回集合对象

Java层的对应方法为：

```java
public class HelloJni {  
    ...  
    //在Native层返回ArrayList集合   
    public native ArrayList<Student> native_getListStudents();  
    ...   
}     
```

Native层该方法实现为 : 

```c++
 */ //获得集合类型的数组  
JNIEXPORT jobject JNICALL Java_com_xxx_jni_HelloJni_native_getListStudents  
  (JNIEnv * env, jobject obj)  
{  
    jclass list_cls = env->FindClass("Ljava/util/ArrayList;");//获得ArrayList类引用  
  
    if(listcls == NULL)  
    {  
        cout << "listcls is null \n" ;  
    }  
    jmethodID list_costruct = env->GetMethodID(list_cls , "<init>","()V"); //获得得构造函数Id  
  
    jobject list_obj = env->NewObject(list_cls , list_costruct); //创建一个Arraylist集合对象  
    //或得Arraylist类中的 add()方法ID，其方法原型为： boolean add(Object object) ;  
    jmethodID list_add  = env->GetMethodID(list_cls,"add","(Ljava/lang/Object;)Z");   
    jclass stu_cls = env->FindClass("Lcom/feixun/jni/Student;");//获得Student类引用  
    
     //获得该类型的构造函数  函数名为 <init> 返回类型必须为 void 即 V  
    jmethodID stu_costruct = env->GetMethodID(stu_cls , "<init>", "(ILjava/lang/String;)V");  
  
    for(int i = 0 ; i < 3 ; i++)  
    {  
        jstring str = env->NewStringUTF("Native");  
        //通过调用该对象的构造函数来new 一个 Student实例  
        jobject stu_obj = env->NewObject(stucls , stu_costruct , 10,str);  //构造一个对象  
          
        env->CallBooleanMethod(list_obj , list_add , stu_obj); //执行Arraylist类实例的add方法，添加一个stu对象  
    }  
  
    return list_obj ;  
} 
```

**Java对象转C++对象流程**
获取Java对象的jclass；

获取Java对象的字段ID，注意字段名称和签名；

根据字段ID获取该字段的值；

JNI类型和C++类型转换；

对C++对象进行赋值；

**C++对象转Java对象流程**
获取Java对象的jclass；

获取构造方法ID；

通过构造方法ID创建Java对象；

获取Java对象的字段ID；

C++类型和JNI类型转换；

通过字段ID给每个字段赋值；


## 二、JNA

### 引入

JNA的引入很方便，使用maven直接导入即可。

```xml
<dependency>
    <groupId>net.java.dev.jna</groupId>
    <artifactId>jna</artifactId>
    <version>5.5.0</version>
</dependency>
```



### 前提条件

JNA有很多条件限制：

1. JNA只能调用C方式编译出来的动态库，若是C++代码则需进行转换。如何用c的方式编译c++动态库，可见链接：[c方式编译c++](https://blog.csdn.net/giveaname/article/details/103133513)
2. 使用中，Java和c/c++的系统版本必须一致，如都是32位或都是64位。



## 使用

扯了这么多，终于要开始调用了。免不了先查询文档：
API：<http://java-native-access.github.io/jna/4.1.0/>
github：<https://github.com/java-native-access/jna>



1. 引入

Jna的样例中，基本都会定义一个接口，该接口链接c/c++动态库，生成一个实例，并定义了与动态库中一致的函数名称，用于后续调用。
举个栗子：