---
title: SOLID原则
author: 
date: 2024-02-06 14:55:00 +0800
categories: [principle]
tags: [cs]
toc: true
---

Michael Feather提出，Robert Martin推广。

## 1. 单一职责原则（Single Responsibility Principle ,SRP）。
一个类应该只有一个修改的原因，也就是说它应该有一个单一的职责或目的。

```go
package main

import "fmt"

type UserManager struct {
	// User management related methods
}

func (um *UserManager) CreateUser(user User) {
	// Logic to create a new user
	fmt.Println("User created successfully!")
}

func (um *UserManager) UpdateUser(user User) {
	// Logic to update an existing user
	fmt.Println("User updated successfully!")
}

type User struct {
	// User related properties
}

func main() {
	userMgr := UserManager{}
	user := User{}
	userMgr.CreateUser(user)
	userMgr.UpdateUser(user)
}
```


## 2. 开闭原则（Open-Close Principle,OCP）。
软件实体（类、模块、函数等）应该对扩展开放，对修改关闭。这意味着您应该能够在不修改现有代码的情况下添加新功能。
```go
package main

import "fmt"

type Shape interface {
	Area() float64
}

type Rectangle struct {
	Width  float64
	Height float64
}

func (r Rectangle) Area() float64 {
	return r.Width * r.Height
}

type Circle struct {
	Radius float64
}

func (c Circle) Area() float64 {
	return 3.14 * c.Radius * c.Radius
}

func CalculateArea(shapes []Shape) {
	for _, shape := range shapes {
		fmt.Println("Area:", shape.Area())
	}
}

func main() {
	shapes := []Shape{
		Rectangle{Width: 5, Height: 10},
		Circle{Radius: 7},
	}
	CalculateArea(shapes)
}
```


## 3. 里氏替换原则（LisKov Substitution Principle,KSP）。
超类的对象应该能够被其子类的对象替换，而不影响程序的正确性。换句话说，子类应该能够在超类的位置上使用，而不会产生任何意外的行为。

```go
package main

import "fmt"

type Shape interface {
	Area() float64
}

type Rectangle struct {
	Width  float64
	Height float64
}

func (r Rectangle) Area() float64 {
	return r.Width * r.Height
}

type Square struct {
	Length float64
}

func (s Square) Area() float64 {
	return s.Length * s.Length
}

func CalculateArea(shape Shape) {
	fmt.Println("Area:", shape.Area())
}

func main() {
	rectangle := Rectangle{Width: 5, Height: 10}
	square := Square{Length: 5}
	CalculateArea(rectangle)
	CalculateArea(square)
}
```
>  在里氏替换原则中，我们遵循"子类对象应该能够替换其父类对象，而不会影响程序的正确性"的原则。换句话说，子类应该能够在不破坏原有系统行为的情况下，替换父类并正常运行。
> 
> 
> 在上述示例中，我们有两个形状：矩形（Rectangle）和正方形（Square）。它们都实现了Shape接口中的Area()方法。
> 
> 
> 在这里，我们使用里氏替换原则来重新设计代码，确保子类（正方形）可以替换父类（矩形）并在不破坏原有系统行为的情况下正常工作。
> 
> 
> 首先，我们定义了Shape接口，其中包含了计算面积的方法Area()。
> 
> 
> 然后，我们实现了Rectangle和Square结构体，它们分别表示矩形和正方形，并且都实现了Shape接口中的Area()方法。请注意，Square结构体重写了Area()方法，以确保其计算面积的逻辑与矩形不同。
> 
> 
> 最后，在main函数中，我们创建了一个Shape类型的切片，并将Rectangle和Square的实例添加到其中。然后，我们通过循环遍历这个切片，并调用每个形状的Area()方法来计算各个形状的面积。
> 
> 
> 通过这种方式，我们可以将正方形对象替换为矩形对象，而不会影响程序的正确性。这是因为在里氏替换原则下，子类应该能够替换父类并保持原有行为的正确性。


## 4. 接口隔离原则（Interface Segreation Principle,ISP）。
客户端不应该被迫依赖它们不使用的接口。与其拥有一个大型接口，不如拥有更小且更专注的接口。
```go
package main

import "fmt"

type Printer interface {
	Print()
}

type Scanner interface {
	Scan()
}

type FaxMachine interface {
	Fax()
}

type MultifunctionDevice interface {
	Printer
	Scanner
	FaxMachine
}

type HPPrinter struct{}

func (p HPPrinter) Print() {
	fmt.Println("Printing...")
}

type CanonScanner struct{}

func (s CanonScanner) Scan() {
	fmt.Println("Scanning...")
}

type EpsonPrinterScanner struct{}

func (ps EpsonPrinterScanner) Print() {
	fmt.Println("Printing...")
}

func (ps EpsonPrinterScanner) Scan() {
	fmt.Println("Scanning...")
}

func main() {
	hpPrinter := HPPrinter{}
	hpPrinter.Print()

	canonScanner := CanonScanner{}
	canonScanner.Scan()

	epsonPrinterScanner := EpsonPrinterScanner{}
	epsonPrinterScanner.Print()
	epsonPrinterScanner.Scan()
}
```
> 在接口隔离原则中，我们遵循"客户端不应该被迫依赖它们不使用的接口"的原则。这意味着我们应该为每个客户端提供最小且专注的接口，以满足其需求，而不强迫客户端依赖于它们不需要的接口。
> 
> 
> 在上述示例中，我们有三个设备：打印机（Printer）、扫描仪（Scanner）和传真机（FaxMachine）。我们还有一个多功能设备（MultifunctionDevice）接口，它继承了这三个设备的接口。
> 
> 
> 在这里，我们使用接口隔离原则来重新设计代码，将多功能设备的功能分离为单独的接口，以便客户端只需依赖于自己需要的接口。这样做的好处是，客户端不再被迫实现或使用它们不需要的方法。
> 
> 
> 首先，我们定义了Printer接口和Scanner接口，分别用于打印和扫描功能。然后，我们定义了FaxMachine接口，用于传真功能。接下来，我们定义了MultifunctionDevice接口，它继承了Printer、Scanner和FaxMachine接口。
> 
> 
> 接下来，我们实现了HPPrinter、CanonScanner和EpsonPrinterScanner结构体，它们分别实现了Printer、Scanner接口。请注意，这些结构体只实现了它们需要的方法。
> 
> 
> 最后，我们在main函数中创建了各种设备的实例，并调用了它们的方法。通过这种方式，我们可以仅使用我们需要的方法，而不会受到其他不必要的方法的干扰。

## 5. 依赖倒置原则（Dependancy Inversion Principle,DIP）。
高层模块不应该依赖于低层模块，两者都应该依赖于抽象。这个原则通过依赖于抽象（接口或抽象类）而不是具体实现，促进了模块之间的松耦合。
```go
package main

import "fmt"

type MessageSender interface {
	SendMessage(message string)
}

type EmailSender struct{}

func (es EmailSender) SendMessage(message string) {
	fmt.Println("Sending email:", message)
}

type SMSWriter struct{}

func (sw SMSWriter) Write(message string) {
	fmt.Println("Sending SMS:", message)
}

type SMSAdapter struct {
	Writer SMSWriter
}

func (sa SMSAdapter) SendMessage(message string) {
	sa.Writer.Write(message)
}

type NotificationService struct {
	Sender MessageSender
}

func (ns NotificationService) SendNotification(message string) {
	ns.Sender.SendMessage(message)
}

func main() {
	emailSender := EmailSender{}
	smsWriter := SMSWriter{}
	smsAdapter := SMSAdapter{Writer: smsWriter}

	notificationService := NotificationService{Sender: emailSender}
	notificationService.SendNotification("Hello, via email!")

	notificationService.Sender = smsAdapter
	notificationService.SendNotification("Hello, via SMS!")
}
```
> 在依赖倒置原则中，我们遵循"高层模块不应该依赖低层模块，两者都应该依赖于抽象"的原则。这意味着我们应该通过抽象来解耦高层和低层模块的依赖关系。


> 在上述示例中，我们有两个类：Client和Service。Client是高层模块，而Service是低层模块。Client依赖于Service来执行一些操作。
> 
> 
> 在这里，我们使用依赖倒置原则来重新设计代码，确保高层模块（Client）依赖于抽象（抽象接口），而不是具体的低层模块（Service）。
> 
> 
> 首先，我们定义了一个抽象接口（IService），它包含了Client所需的操作方法。
> 
> 
> 然后，我们修改了Service类，使其实现IService接口。这样，Service类就成为了抽象接口的具体实现。
> 
> 
> 接下来，我们在Client类中使用构造函数来注入一个IService类型的对象。通过这种方式，Client类不再直接依赖于具体的Service类，而是依赖于抽象接口。
> 
> 
> 最后，在main函数中，我们实例化了Client类，并将具体的Service对象传递给它。这样，我们就将具体的依赖关系转移到了对象的创建过程中，而不是在Client类内部创建。


通过这种方式，我们实现了依赖倒置原则，使得高层模块（Client）依赖于抽象接口（IService），而不是具体的低层模块（Service）。这样做的好处是，我们可以轻松地替换具体的低层模块，而不会影响高层模块的实现。

