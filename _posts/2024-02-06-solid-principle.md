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