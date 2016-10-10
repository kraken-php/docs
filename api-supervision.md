# Supervision Component

- [Introduction](#introduction)
- [Features](#features)
- [Concepts](#concepts)
- [Examples](#examples)
    - [Registering Problem Solvers](#registering-problem-solvers)
    - [Registering Multiple Solvers](#registering-multiple-solvers)
    - [Solving Problems](#solving-problems)
    - [Solvers Resolution](#solvers-resolution)
    - [Writing Custom Solvers](#writing-custom-solvers)
- [Important Classes & Interfaces](#important-classes-and-interfaces)
    - [Solver](#solver)
    - [SolverComposite](#solver-composite)
    - [SolverInterface](#solver-interface)
    - [SolverFactory](#solver-factory)
    - [SolverFactoryInterface](#solver-factory-interface)
    - [Supervisor](#supervisor)
    - [SupervisorInterface](#supervisor-interface)

<a name="introduction"></a>
## Introduction

Supervision is component that provides supervision mechanism for runtime containers and common problem solvers.

<a name="introduction"></a>
## Features

Supervision features:

<div class="dot-list" markdown="1">
- Error supervisor
- Implementation of simple and composite solvers
- Kraken Framework compatibility
</div>

<a name="concepts"></a>
## Concepts

This section contains terminology, useful concepts and definitions, that might be helpful to learn to fully understand this component purpose.

### Supervison

Supervision is dependency relationship between supervisor and supervised code, in which supervisor must respond to its supervised code execution failures. When a failure is detected, the supervisor should stop execution of supervised code and react accordingly. Supervision systems are usually used in distributed systems, in which parent process supervises its children. 

### Supervisor

Supervisor is an object that supervises part of code or application architecture.

### Solver

Solver is an abstraction of a measure that have to be taken to solve problem which occurred.

<a name="examples"></a>
## Examples

This section contains examples and patterns that can be used with described component.

<a name="registering-problem-solvers"></a>
### Registering Problem Solvers

Registering problem solvers can be done via passing its instance or class name to supervisor constructor:

    $supervisor = new Supervisor($factory, [], [
        CustomException::class => CustomSolver::class
    ]);

Or by passing adding it dynamically using `setSolver` method.

    $supervisor->setSolver(CustomException::class, CustomSolver::class);

<a name="registering-multiple-solvers"></a>
### Registering Multiple Solvers

Multiple solvers for one problem can be also defined using array syntax:

    $supervisor = new Supervisor($factory, [], [
        CustomException::class => [ FirstSolver::class, SecondSolver::class ]
    ]);

<a name="solving-probles"></a>
### Solving Problems

Solving problems can be used by calling `solve` method.

    try
    {
        // some code
    }
    catch (Throwable $throwable)
    {
        $supervisor->solve($throwable);
    }

<a name="solvers-resolution"></a>
### Solvers Resolution

> {warning} Supervisor `solve` methods behaves the same way as declaring multiple `catch` blocks. This means that the first valid solver will be the only one executed, so pay attention to the order of your solvers.

    $supervisor = new Supervisor($factory, [], [
        Error::class            => ASolver::class,
        Exception::class        => BSolver::class,
        CustomException::class  => CSolver::class
    ]);
    
    // since CustomException is instance of Exception, the first and the only solver to be execute will be BSolver
    $supervisor->solve(new CustomException());

> {tip} It is a good idea to register `Throwable::class` solver as the last one which will behave in similar way that `default` does in switches.

<a name="writing-custom-solvers"></a>
### Writing Custom Solvers

All of custom solvers need to extend `Kraken\Supervision\Solver` class and implement `Kraken\Supervision\SolverInterface` interface.

    use Kraken\Supervision\Solver;
    use Kraken\Supervision\SolverInterface;
    
    class CmdDoNothing extends Solver implements SolverInterface
    {
        protected function solver($ex, $params = [])
        {
            // this code will be executed and promisified by Supervisor
        }
    }

<a name="important-classes-and-interfaces"></a>
## Important Classes & Interfaces

This section contains list of most important classes and interfaces shipped with this component. It **does not include all** classes and interface.

<a name="solver"></a>
### Solver

    class Solver implements SolverInterface

Solver represents the handler that is executed as a response for thrown error or exception which intention is to solve a problem which occurred.

<a name="solver-composite"></a>
### SolverComposite

    class SolverComposite implements SolverInterface

SolverComposite is a class that allows creating chain of solvers which are executed one after another. It is important to keep in mind, that if one of solvers in chain fails then the rest of them won't be executed, as the overall resolution of problem will be marked as failure.

<a name="solver-interface"></a>
### SolverInterface

    interface SolverInterface

<a name="solver-factory"></a>
### SolverFactory

    class SolverFactory extends Factory implements SolverFactoryInterface

SolverFactory is the factory that knows how to create all of solvers used in application.

<a name="solver-factory-interface"></a>
### SolverFactoryInterface

    interface SolverFactoryInterface extends FactoryInterface

<a name="supervisor"></a>
### Supervisor

    class Supervisor implements SupervisorInterface

Supervisor is an instance of an object that that supervises execution of your code and in case of failure, executes proper solvers.

<a name="supervisor-interface"></a>
### SupervisorInterface

    interface SupervisorInterface
