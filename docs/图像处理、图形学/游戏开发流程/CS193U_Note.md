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


## GamePlay Collision & Physics

### Character Input & Rotations

The Character's input needs improvement. First of all, we need to set the ControlRotation to true. And there are some notion need to be clear, which are **pitch**, **yaw** and **roll**. 

In UE system, axis **X** means **forward**, axis **Y** means **right** and axis **Z** means **up**. In this coordinate system, the above several concepts are the Euler angles in the three-dimensional world. **Pitch** is the angle rotates around the axis **Y**, **Yaw** is the angle rotates around the axis **Z**, and **Roll** is the angle rotates around the axis **X**.

### Magic Projectile Attack

This section is about the attck part of the character. The character is able to attck due to the projectile module. 

First of all, we need the mesh of the projectile, and we find the character's skeleton and bind the projectile to the hand of the character. 

### Some Function Notion

``` c++ linenums="1"
FORCEINLINE class UCharacterMovementComponent* GetCharacterMovement() const;
```

> **UCharacterMovementComponent**封装了大量的移动相关功能，例如行走、跑步、跳跃、坠落、飞行和游泳等等。并且也融合了很多的高级物理模拟功能

``` c++ linenums="1"
AddMovementInput(GetActorForwardVector(), value);

auto ControlRot = GetControlRotation();
ControlRot.Pitch = 0.0f;
ControlRot.Roll = 0.0f;
AddMovementInput(FRotationMatrix(ControlRot).GetScaledAxis(EAxis::Y);
```

> **FRotationMatrix**是根据你的输入情况得到的对应欧拉角角度变换差。而**GetActorForwardVector**是角色的向量角，使用这个函数会导致当我们向左运动时，会不停地原地旋转。我们把Pitch和Roll设置为零，就仅考虑Yaw角度即可。

``` c++ linenums="1"
inline USKeletalMeshComponent* ACharacter::GetMesh() const;

virtual FVector USceneXomponent::GetSocketLocation(FName InSocketName) const;

FActorSpawnParameters SpawnParams;
SpawnParams.SpawnCollisionHandlingOverride = 
    ESpawnActorCollisionHandlingMethod::AlwaysSpawn;

virtual UWorld* AActor::GetWorld() const override;
inline AActor* UWorld::SpawnActor<AActor>(UClass* class, const FTransform, const FActorSpawnParameters &SpawnParameters);
```

> **GetMesh**用来获取这个网状网络（通常就是预先做好的人物模型、物体模型等等）
> **GetSocketLocation**用来获得制定部位的位置，输出为向量格式。上面两步就是用来获得手部（或者其他的你想要设定的位置）的位置
> **FTransform**用来记录Actor的位置和旋转方向
> **FActorSpawnParameters**中，这个override参数是用来将Projectile无条件发射出去。即便人物的手已经在墙体里了，也不会消失或者被阻挡。
> 最终使用**GetWorld**函数获取当前游戏上下文，然后使用**SpawnActor**生成一个AActor实例，传入函数需要的旋转、位置参数、物理碰撞参数和物体的类型。
> 其中需要特殊说明的是：**TSubclassOf**这个类是在创建一个类的子类时经常用到的。其相比UClass，可以更加显式地指明这是什么类，并且说明了其父类是什么。

## Interfaces & Collision Queries

### C++ Interfaces

**Unreal Interfaces** is been made for general instrument, such as interactions, functions, team work and so on. It's aim is providing services for diverse components by using unified method. 

When we choose **Unreal Interfaces**, UE will generate two class. One starts with U and the other starts with I. **I** is for dll export, it is the real functioning module. **U** aims at being detected by the Unreal Engine's *Reflection System*.

Its logic is, you need to inherit this class when you are to use this interactions. Then we could use its member virtual function.

### Some Function Notion

``` c++ linenums="1"
ISGamePlayInterface::Execute_Interact(UObject* O, Apawn* InstigatorPawn);
```

> **Execute_Interact**是UE用来给蓝图设计的投影函数。如果是仅仅在C++的多态环境中，仅需要调用自己创建的ISGamePlayInterface类中的 Interact 成员函数即可。但是在蓝图中，这个函数就会失效，即程序是看不见的。这是就需要UE中的reflection function来产生一个中间过程的函数 Execute_Interact。首先其会判断在蓝图中有没有这个接口函数，如果没有那么就会直接调用C++类中的成员函数。

``` c++ linenums="1"
AActor* MyOwner = GetOwner();
FVector EyeLocation; //This may not the correct position in 3rd Game
FRotator EyeRotation;
MyOwner->GetActorEyesViewPoint(EyeLocation, EyeRotation);
auto End = EyeLocation + (EyeRotation.Vector() * 1000);

FCollisionObjectQueryParams ObjectQueryParams;
ObjectQueryParams.AddObjectTypesToQuery(ECollisionChannel::ECC_WorldDynamic);

constexpr float Radius = 30.0f;
FCollisionShape Shape;
Shape.SetSphere(Radius);

TArray<FHitResult> Hits;
bool IfSphereHit = GetWorld()->SweepMultiByObjectType(
	Hits,
	EyeLocation, End,
	FQuat::Identity,
	ObjectQueryParams,
	Shape
);
FColor LineColor = IfSphereHit ? FColor::Green : FColor::Red;

for (const auto& Hit : Hits) {
	AActor* HitActor = Hit.GetActor();
	if (HitActor != nullptr) {
		if (HitActor->Implements<USGamePlayInterface>()) {
			APawn* MyPawn = Cast<APawn>(MyOwner);
			ISGamePlayInterface::Execute_Interact(HitActor, MyPawn);
			break;
		}
	}

	DrawDebugSphere(GetWorld(), Hit.ImpactPoint, Radius, 32, LineColor, false, 20.f);
}

/*Draw debug*/
DrawDebugLine(GetWorld(), EyeLocation, End, LineColor, false, 2.0f, 0, 2.0f);
```

> 这是一段经典的交互场景下的代码。首先通过确定眼睛的位置来判断我们的物体是否在我们鼠标或者人物朝向的方向上，然后通过判断这条线上是否存在一个物体（原理类似于光线追踪）。若物体存在，则就可以进行交互操作
> **GetActorEyesViewPoint**即我们确定眼睛的向量函数，1000是距离，确定线段的终点在哪里
> 但是这样子的判断是一个点，很多时候，交互的物体会很小。此时可以使用**SweepMultiByObjectType**函数来判断在一个圆球中，是否有物体碰撞。其中**ECC_WorldDynamic**这个参数是用来确定物体的query性质，即我们的物体是动态移动的。回到**SweepMultiByObjectType**这个函数，该函数最终可以获得一堆在圆球中的物体，但是我们仅仅是需要一个，所以在for循环中，我们只要获取到了一个，就会break掉，跳出循环，而不是将所有的物体都进行互动，当然也可以根据设计需要进行调整
> **Implements**确定了这个物体是否是可交互的。然后使用**Execute_Interact**，传入玩家（Instigator），让物体根据操作定义来进行动作。


