# CS193U Note

The course's Link: [Tom Looman](https://courses.tomlooman.com)

## Introduction & Setup

- Platform: MacOS or Windows 
- Tools: Rider, Visual Studio, Visual Studio Code, Unreal Engine
- Version: Visual Studio (2019), Unreal Engine(4.27) 

> *PS: only need the generation tool with the version of 2019. The VS2019's Link: [Visual Studio 2019](https://pan.baidu.com/s/1ML-Yf2-r6cQKNp1avORFvA?pwd=frhu). If you are using VS2026, first you need to install VS2022 generation tool, subsequently you could install the generation tool with version of VS2019.*

There are bunch of name specification need to be clear:
- **U** : 引擎对象，UE反射系统的基础，支持垃圾回收、序列化等等
- **A** : 演员，可以放置在场景中，拥有各种图形学变换的功能支持
- **F** : 普通结构体，不具备反射系统，纯数据结构，内存管理需要C++的支持
- **I** : 接口，定义了一组行为规范，不包含具体实现 

## Component Adding

For a simple component, we need to add two Class's declaration and instantiation into the **Class ALinglongCharacter** :

``` c++ title="LinglongCharacter.h" linenums="1"
UPROPERTY(VisibleAnywhere)
USpringArmComponent* _spring_arm_comp;

UPROPERTY(VisibleAnywhere)
UCameraComponent* _camera_comp;
```

Obviously, these are two member varaible. There are something need to be explained:

- The name of the **Class ALinglongCharacter**: *Linglong* is the custom name of this component. *A* are one of the UE's class prefix, which represents deriving from *Actor*.
- Why these two? SpringArm is the tool of the camera component. It's widely used in 3rd person game, which is similar to the camera arm. This component is able to follow the charactor and avoid the wall and the ground collison problem. The camera component is the Camera of the charactor which is bound to the SpringArm. 
- What is UPROPERTY? Why can't I find the definition in C++ file? **UPROPERTY** is the marco definition in UE, however it couldn't be found in C++. When it has been compiled, it would be stright remain as **UPROPERTY**, then will be recognized by UE, and this Class will become visible detemined by the varaible within the bracket. 

And there is also the code of the instantiation of the member: 

``` c++ title="LinglongCharacter.cpp" linenums="1"
this->_spring_arm_comp = CreateDefaultSubobject<USpringArmComponent>("spring_ arm_comp");
this->_camera_comp = CreateDefaultSubobject<UCameraComponent>("camera_comp");

this->_spring_arm_comp->SetupAttachment(RootComponent);
this->_camera_comp->SetupAttachment(this->_spring_arm_comp);
```

These two code achieved the instantiation and the attachment of two components. 

### Some Function Notion

``` c++ title="Object.h" linenums="1"
template<class TReturnType>
TReturnType* CreateDefaultSubobject(FName SubobjectName, bool bTransient = false);
```

> 这段代码是使用了C++一种泛型编程的应用。**UClass**是一种C++元数据结构，在UE执行过程中会被识别，并且自动推导是哪一种子类。目的是完善了C++中的类型识别（RTTI）。  
> 在预编译过程中，当**Unreal Header Tool**看到这个宏定义的时候，就会识别到是什么类型的类或者函数，然后生成对应的代码。

``` c++ title="SceneComponent.cpp" linenums="1"
void USceneComponent::SetupAttachment(class USceneComponent* InParent, FName InSocketName);
```

> 该函数可以处理组件之间的所属关系，是**USceneComponent**类的成员函数。**USceneComponent**具备空间转换能力，很多的相机、弹簧臂、静态Mesh都是其子类。主要的功能就是确定空间中Level中的组件位置关系。

``` c++ linenums="1"
void APawn::AddMovementInput(FVector WorldDirection, float ScaleValue, bool bForce);

void UPawnMovementComponent::AddInputVector(FVector WorldAccel, bool bForce);

void APawn::Internal_AddMovementInput(FVector WorldAccel, bool bForce);

FORCEINLINE_DEBUGGABLE FVector AActor::GetActorForwardVector()
```

> **AddMovementInput**赋予了一个Actor行动的能力，根据输入的向量，来确定这个人物的行动。**AddInputVector**则是输入我们想要的向量。  
> 在**AddInputVector**这个函数中，比较有意思的点在于，这个所属类的Prefix是**U**，但是这个类保存了其所属的**APawn**类，所以二者是友元关系。  
> 最终在处理这个向量的过程中，**APawn**类中保存了成员变量*ControlInputVector*这个基础上加上我们的输入。这个原因在于在同一时间，会有很多个向量的输入，所以为了综合考量这个输入，所以就使用了向量加法，构成最终的向量。

