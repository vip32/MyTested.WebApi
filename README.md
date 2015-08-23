# MyWebApi - ASP.NET Web API Fluent Testing Framework
------------------------------------

MyWebApi is unit testing framework providing easy fluent interface to test the ASP.NET Web API framework. Inspired by [TestStack.FluentMVCTesting](https://github.com/TestStack/TestStack.FluentMVCTesting) and [ChaiJS](https://github.com/chaijs/chai)

[![Build status](https://ci.appveyor.com/api/projects/status/738pm1kuuv7yw1t5?svg=true)](https://ci.appveyor.com/project/ivaylokenov/mywebapi)

## How to use

### Controller instantiation

You have a couple of options from which you can setup the controller you want to test. The framework gives you static `MyWebApi` class from which the test builder starts:

```c#
// instantiates controller with parameterless constructor
MyWebApi
	.Controller<WebApiController>();
	
// instantiates controller with constructor function to resolve dependencies
MyWebApi
	.Controller(() => new WebApiController(mockedInjectedService));
```

### Calling actions

You can call any action using lambda expression. All parameter values will be resolved and model state validation will be performed on them:

```c#
// calls action with no parameters
MyWebApi
	.Controller<WebApiController>()
	.Calling(c => c.SomeAction());
	
// calls action with parameters
MyWebApi
	.Controller<WebApiController>()
	.Calling(c => c.SomeAction(requestModel));

// calls async action
MyWebApi
	.Controller<WebApiController>()
	.CallingAsync(c => c.SomeActionAsync());
```

### Model state validation

You can test whether model state is valid/invalid or contains any specific error:

```c#
// tests whether model state is valid
MyWebApi
	.Controller<WebApiController>()
	.Calling(c => c.SomeAction(requestModel))
	.ShouldHaveValidModelState();
	
// tests whether model state is not valid
MyWebApi
	.Controller<WebApiController>()
	.Calling(c => c.SomeAction(requestModel))
	.ShouldHaveInvalidModelState();
	
// tests whether model state error exists (or does not exist) for specific key (not recommended because of magic string)
MyWebApi
	.Controller<WebApiController>()
	.Calling(c => c.SomeAction(requestModel))
	.ShouldHaveModelStateFor<RequestModel>()
	.ContainingModelStateError("propertyName")
	.And() // calling And method is not necessary but provides better readability
	.ContainingNoModelStateError("anotherPropertyName");
	
// tests whether model state error exists by using lambda expression
MyWebApi
	.Controller<WebApiController>()
	.Calling(c => c.SomeAction(requestModel))
	.ShouldHaveModelStateFor<RequestModel>()
	.ContainingModelStateErrorFor(m => m.SomeProperty)
	.And()
	.ContainingNoModelStateErrorFor(m => m.AnotherProperty);
	
// tests the error message for specific property
MyWebApi
	.Controller<WebApiController>()
	.Calling(c => c.SomeAction(requestModel))
	.ShouldHaveModelStateFor<RequestModel>()
	.ContainingModelStateErrorFor(m => m.SomeProperty).ThatEquals("Error message") // error message must be equal to the provided string
	.And()
	.ContainingModelStateErrorFor(m => m.SecondProperty).BeginningWith("Error") // error message must begin with the provided string
	.And()
	.ContainingModelStateErrorFor(m => m.ThirdProperty).EndingWith("message") // error message must end with the provided string
	.And()
	.ContainingModelStateErrorFor(m => m.SecondProperty).Containing("ror mes"); // error message must contain the provided string
```

### Action results

You can test for specific return values or the default IHttpActionResult types:

#### Generic result

Useful where the action does not return IHttpActionResult

```c#
// tests whether the action returns certain type by providing generic parameter
MyWebApi
	.Controller<WebApiController>()
	.Calling(c => c.SomeAction())
	.ShouldReturn<ResponseModel>();
	
// tests whether the action returns certain type by using typeof
MyWebApi
	.Controller<WebApiController>()
	.Calling(c => c.SomeAction())
	.ShouldReturn(typeof(ResponseModel));
	
// tests whether the action returns generic model
MyWebApi
	.Controller<WebApiController>()
	.Calling(c => c.SomeAction())
	.ShouldReturn(typeof(IList<>)); // works with IEnumerable<> (or IList<ResponseModel>) too by using polymorphism
```

#### Ok result

```c#
// tests whether the action returns OkResult
MyWebApi
	.Controller<WebApiController>()
	.Calling(c => c.SomeAction())
	.ShouldReturnOk();
	
// tests whether the action returns OkResult with no response model
MyWebApi
	.Controller<WebApiController>()
	.Calling(c => c.SomeAction())
	.ShouldReturnOk()
	.WithNoResponseModel();
	
// tests whether the action returns OkResult with specific response model type
MyWebApi
	.Controller<WebApiController>()
	.Calling(c => c.SomeAction())
	.ShouldReturnOk()
	.WithResponseModel<ResponseModel>();
	
// tests whether the action returns OkResult with specific object
MyWebApi
	.Controller<WebApiController>()
	.Calling(c => c.SomeAction())
	.ShouldReturnOk()
	.WithResponseModel(someResponseModelObject);

// tests whether the action returns OkResult with specific response model passing certain assertions
MyWebApi
	.Controller<WebApiController>()
	.Calling(c => c.SomeAction())
	.ShouldReturnOk()
	.WithResponseModel<ResponseModel>(m =>
	{
		Assert.AreEqual(1, m.Id);
		Assert.AreEqual("Some property value", m.SomeProperty);
	});
	
// tests whether the action returns OkResult with specific response model passing a predicate
MyWebApi
	.Controller<WebApiController>()
	.Calling(c => c.SomeAction())
	.ShouldReturnOk()
	.WithResponseModel<ResponseModel>(m => m.Id == 1);
	
// tests for model state errors for the response model (not very useful in practice)
MyWebApi
	.Controller<WebApiController>()
	.Calling(c => c.SomeAction())
	.ShouldReturnOk()
	.WithResponseModel<ResponseModel>()
	.ContainingModelStateErrorFor(m => m.SomeProperty).ThatEquals("Error message")
	.And()
	.ContainingNoModelStateErrorFor(m => m.AnotherProperty);
```

## Any questions, comments or additions?

Leave an issue on the [issues page](https://github.com/ivaylokenov/MyWebApi/issues) or send a [pull request](https://github.com/ivaylokenov/MyWebApi/pulls).