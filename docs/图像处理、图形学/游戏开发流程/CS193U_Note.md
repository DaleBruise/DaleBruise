# CS193U Note

The course's Link: [Tom Looman](https://courses.tomlooman.com)

## Introduction & Setup

- Platform: MacOS or Windows 
- Tools: Rider, Visual Studio, Visual Studio Code, Unreal Engine
- Version: Visual Studio (2019), Unreal Engine(4.27) 

> *PS: only need the generation tool with the version of 2019. The VS2019's Link: [Visual Studio 2019](https://pan.baidu.com/s/1ML-Yf2-r6cQKNp1avORFvA?pwd=frhu). If you are using VS2016, first you need to install VS2022 generation tool, subsequently you could install the generation tool with version of VS2019.*

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

