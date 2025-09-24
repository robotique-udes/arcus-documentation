# CPP Coding Guidelines

Guidelines are there to help organise and increase readability and maintainability. The following are guidelines, not strict rules, you should always try to conform to them but they can always be broken if it increase readability or maintainability. It's a constant work in progress and it's opened to suggestions

## CPP Naming Style

| Type of data         | Styling               | Example      |
| -------------------- | --------------------- | ------------ |
| variable             | camelCase             | `myInt`      |
| function name        | camelCase             | `myFunc()`   |
| class and namespaces | PascalCase            | `MyClass`    |
| Macro                | UPPER_CASE_SNAKE_CASE | `MY_MACRO()` |
| Constant and define  | UPPER_CASE_SNAKE_CASE | `PI`         |
| Enum elements        | UPPER_CASE_SNAKE_CASE | `FIRST_ELEM` |

### Prefix and suffix

| Type of data            | Styling         | Example            |
| ----------------------- | --------------- | ------------------ |
| Argument name           | camelCase\_     | `myArg_ `          |
| Non Const Reference     | rPascalCase\_   | `rMyRef`           |
| private member variable | \_camelCase     | `_myPrivateMember` |
| global variable         | g_camelCase     | `g_myInt`          |
| Struct                  | sPascalCase     | `sMyStruct`        |
| Enum                    | ePascalCase     | `eMyEnum`          |
| ROS Subscriber          | sub_camelCase   | `sub_mySrg`        |
| ROS Publisher           | pub_camelCase   | `pub_mySrg`        |
| ROS Timer               | timer_camelCase | `timer_mySrg`      |

## Use Allman brackets style

### do:

```cpp
if (something)
{
  doSomethingElse();
}
```

### avoid:

```cpp
if (something) {
  doSomethingElse();
}
```

## Do not use dynamic allocation on microcontrollers

Using dynamic allocation can fragment the heap memory ([more info](https://stackoverflow.com/questions/3770457/what-is-memory-fragmentation)), it's generally always a bad idea on systems which requires a high level of reliability for long periods of time with low memory footprint. For this reason, the use of the _new_, _delete_, _std::share_ptr_, _std::unique_ptr_, etc. is discouraged on the rover. The std library uses a lot of dynamic allocation so it's generally a bad idea to use it.

### Do:

```cpp
int8_t myInt = 8;
int8_t* myIntPtr = &myInt;

something(&myInt);
```

```cpp
std::array<int, 3> myArray(1, 2, 3); // std::array is statically allocated
printf("%i", myArray[2]);
```

### Avoid:

```cpp
int8_t* myInt = new int; // Using dynamic allocation
*myInt = 6;

something(myInt);
```

```cpp
std::vector<int> myVector; // Vector is dynamically allocated
myVector.append(1);
myVector.append(2);
myVector.append(3);
```

On ROS code it's tolerated when it make the code easier to maintain and easier to read. ROS is also full of dynamic allocation and the rover computer has 16gb of RAM.

## Use full namespace

It might seems harder to read at first, but using full namespaces increase readability when you didn't write the code.

### Do:

```cpp
#include "rover_can_lib/rover_can_lib.hpp"
[...]
    if (canBus.isOk() && msgPropCmd.data.enable)
    {
        if (DEVICE_ID == (uint16_t)RoverCanLib::Constant::eDeviceId::REARRIGHT_MOTOR)
        {
            speedCmd = cmdAverage.addValue(msgPropCmd.data.targetSpeed * -100.0f);
        }
    }
[...]
```

### Avoid:

```cpp
#include "rover_can_lib/rover_can_lib.hpp"

using RoverCanLib::Constant::eDeviceId;

[...]
    if (canBus.isOk() && msgPropCmd.data.enable)
    {
        if (DEVICE_ID == (uint16_t)REARRIGHT_MOTOR)
        {
            speedCmd = cmdAverage.addValue(msgPropCmd.data.targetSpeed * -100.0f);
        }
    }
[...]
```

## Use the _this->_ pointer when calling a method from inside it's own class

### do:

```cpp
// inside a class
calculateSpeed(this->getPosition());
```

### Avoid:

```cpp
// inside a class
calculateSpeed(getPosition()); // It's ambiguous if getPosition is a method from the class or a global function
```

## Use .hpp and .cpp file extension if using cpp feature

Example for a class named MyClass

```cpp
my_class.hpp
my_class.cpp
```

## Enum should always be of type _enum class_

Enum class allows users to scope enums and force the use of enum members from their namespace

### do:

```cpp
enum class eDataIndex : uint8_t // Forcing a enum size (by it's type) is a good practice
{
    MSG_ID = 0x00,
    MSG_CONTENT_ID = 0x01,
    START_OF_DATA = 0x02
};

uint8_t msg[MSG_LENGTH] = {0};
msg[(uint8_t)eDataIndex::MSG_CONTENT_ID] = someDeviceId;
```

### avoid:

```cpp
enum eDataIndex : uint8_t
{
    MSG_ID = 0x00,
    MSG_CONTENT_ID = 0x01,
    START_OF_DATA = 0x02
};

uint8_t msg[MSG_LENGTH] = {0};
msg[MSG_CONTENT_ID] = someDeviceId; // Where MSG_CONTENT_ID is ambiguous
```

## Enum class should only be casted into their underlying _int_ type

### do:

```cpp
enum class eDataIndex : uint8_t // Defining type: uint8_t
{
    MSG_ID = 0x00,
    MSG_CONTENT_ID = 0x01,
    START_OF_DATA = 0x02
};

uint8_t msg[MSG_LENGTH] = {0};
msg[(uint8_t)eDataIndex::MSG_CONTENT_ID] = someDeviceId; // Casting into uint8_t
```

### avoid:

```cpp
enum class eDataIndex : uint8_t // Defining type: uint8_t
{
    MSG_ID = 0x00,
    MSG_CONTENT_ID = 0x01,
    START_OF_DATA = 0x02
};

uint8_t msg[MSG_LENGTH] = {0};
msg[(uint16_t)eDataIndex::MSG_CONTENT_ID] = someDeviceId; // Casting into uint16_t, undefined behavior
```

## Macros

Macro can be written using either #defines or the constexpr keyword. The constexpr keyword is generally suggested as it's safer, easier to write and to read. But both can be used:

### example:

```cpp
#define SQUARE(x) ((x) * (x))

constexpr int SQUARE(int x)
{
    return x * x;
}
```

## Instead of using the const keyword, either use define or constexpr

[great explanation here](https://stackoverflow.com/questions/13346879/const-vs-constexpr-on-variables).
Constexpr are suggested over #define because they can be scoped inside class or namespace which make their use less ambiguous. They also have a type which make type mistakes less likely then defines. Both are still tolerated and used in our codebase.

### do:

```cpp
#define PHIL

namespace KEYBINDING
{
#if defined (PHIL) // define are great for preprocessor directives
    constexpr uint8_t JOINT_SELECT_INC = rover_msgs::msg::Joy::CROSS_UP; // can be access only by it's namespace: KEYBINDING::JOINT_SELECT_INC
    constexpr uint8_t JOINT_SELECT_DEC = rover_msgs::msg::Joy::CROSS_DOWN;
[...]
}
```

### avoid:

```cpp
#define JOINT_SELECT_DEC rover_msgs::msg::Joy::CROSS_DOWN; // Tolerated
const uint8_t JOINT_SELECT_INC = rover_msgs::msg::Joy::CROSS_UP; // Don't
```

## Use available logger instead of normal printf/Serial.print

### do:

```cpp
// On microcontroller projects:
LOG(INFO, "Printing a number: %u as info", 4u);

// On ROS projects inside a class of *Node* type:
RCLCPP_INFO(this->get_logger(), "Printing a number %u as info", 4u);

// On ROS projects outside a class of *Node* type:
RCLCPP_INFO(rclcpp::get_logger("Relevant Name"), "Printing a number %u as info", 4u);
```

### avoid:

```cpp
// On microcontroller projects:
Serial.printf("Printing a number: %u", 4u);

// On ROS projects
printf("Printing a number %u", 4u);
```

## Use curly brackets for one line nested block

### do:

```cpp
if (you.hasAnswer())
{
    you.postAnswer();
}
else
{
    you.doSomething();
}
```

### avoid:

```cpp
if (you.hasAnswer())
    you.postAnswer();
else
    you.doSomething();
```

## Code shouldn't overshoot the 130 character line

When breaking on multiple line, make sure all elements take only one line. For example don't put 1 argument on the first line and 2 on the second. Use 3 lines to one for each arguments

### do:

```cpp
                                                                                                                     130
CanDevice(uint16_t id_,                                                                                                |
          CanMaster *canMasterPtr_,                                                                                    |
          void (CanMaster::*callback_)(uint16_t id_, const can_frame *frameMsg_),                                      |
          rclcpp::Publisher<rover_msgs::msg::CanDeviceStatus>::SharedPtr pub_CanBusState_)                             |
```

### avoid:

```cpp
                                                                                                                     130
CanDevice(uint16_t id_, CanMaster *canMasterPtr_, void (CanMaster::*callback_)(uint16_t id_, const can_frame *frameMsg_|), rclcpp::Publisher<rover_msgs::msg::CanDeviceStatus>::SharedPtr pub_CanBusState_)
                                                                                                                       |
```

To add the line in your vscode do the following:

- ctrl+shift+p
- Search for: "Preferences: Open User Settings (JSON)
- Add these lines as a new entry:

```JSON
    "editor.rulers": [
        130
    ],
```

## Prefer lambda function over std::bind

Lambda function are often simpler and use less overhead.

### do:

```cpp
_sub_joyArm = this->create_subscription<rover_msgs::msg::Joy>("/rover/arm/joy",
                                                              1,
                                                              [this](const rover_msgs::msg::Joy::SharedPtr joyMsg_)
                                                              { this->CB_joy(joyMsg_); });
```

### avoid:

```cpp
_sub_joyArm = this->create_subscription<rover_msgs::msg::Joy>("/rover/arm/joy",
                                                              1,
                                                              std::bind(&Teleop::CB_joy, this, std::placeholders::_1));
```

## Prefer passing by reference when pointer can't be null

Makes it simpler and safer

### do:

```cpp
void myFunc(float &rValue)
{
  rValue += 2;
  return;
}
```

### avoid:

```cpp
void myFunc(float *pValue)
{
  if (pValue)
  {
    *pValue += 2;
  }
  return;
}
```

## Class header styling

Not optimal for compilation time but nicer for highlighting constants

### do:

```cpp
class <NAME>
{
public:
	<PUBLIC STATIC CONSTEXPR>
private:
	<PRIVATE STATIC CONSTEXPR>

public:
	<PUBLIC STRUCT/ENUM/UNION/ETC.>
private:
	<PRIVATE STRUCT/ENUM/UNION/ETC.>

public:
	<CONSTRUCTORS>

	<PUBLIC METHODS>

private:
	<PRIVATE METHODS>

	<MEMBERS>
}
```

### example:

```cpp
class ArucoDetectionNode : public rclcpp::Node
{
    static constexpr uint16_t DELAY_PUBLISHER_MS = 1000;
    static constexpr uint16_t DELAY_DETECTION_MS = 50;

  public:
    ArucoDetectionNode(void);

  private:
    void CB_aruco_publisher(void);
    void CB_aruco_detection(void);

    rclcpp::Publisher<rover_msgs::msg::Aruco>::SharedPtr _publisher;
    rclcpp::TimerBase::SharedPtr _timerPublisher;
    rclcpp::TimerBase::SharedPtr _timerDetection;

    std::unique_ptr<Detection> _detection;
    std::mutex _mutex;
};
```

## One declaration per line

Better readability and prevent mistakes

### do:

```cpp
int a = 1;
int b = 3;
```

### avoid:

```cpp
int a = 1, b = 3;
```

## Include order

Local includes (with quotes `""`) should be before external includes `<>`.

### do:

```cpp
#include "some_project_file.hpp"

#include <rover_lib2/helpers/macros.hpp>
#include <rclcpp/rclcpp.h>
#include <vector>
```

### avoid:

```cpp
#include <rover_lib2/helpers/macros.hpp>
#include <rclcpp/rclcpp.h>
#include <vector>

#include "some_project_file.hpp"
```do
