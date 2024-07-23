
# Results

`ChrisMavrommatis.Results` is a .NET library that provides utility types for handling results and union types in a clean and expressive manner.

## Installation
You can install the package via NuGet Package Manager:
```cmd
Install-Package ChrisMavrommatis.Results
```
Or via .NET CLI:
```cmd
dotnet add package ChrisMavrommatis.Results
```

## Usage
The library contains two main result types:
- `Result<T>`
- `OneOf<...>`

## Result

The `Result<T>` class is used to represent the outcome of an operation, encapsulating both success and failure states while containing the successgull result's object.

### Creating a Successful Result
```csharp
var successResult = Result<int>.Successful(42);
Console.WriteLine(successResult.Success); // True
Console.WriteLine(successResult.Value);   // 42
```

### Creating a Failed Result
```csharp
var failedResult = Result<int>.Failed("An error occurred");
Console.WriteLine(failedResult.Success);  // False
Console.WriteLine(failedResult.Message);  // "An error occurred"
```

## Void Result
The `Result` class is used to represent the outcome of an operation, encapsulating both success and failure states but without containing an obejct, which is usefull for operation that don't have a result.

## OneOf
The `OneOf<...>` struct is used to represent a value that can be one of N possible types.

Suported OneOf types
- `OneOf<T0, T1>`
- `OneOf<T0, T1, T2>`
- `OneOf<T0, T1, T2, T3>`

### Creating a OneOf Instance
```csharp
public async Task<OneOf<User, Conflict, Error>> CreateAsync(CreateUserRequest request, CancellationToken cancellationToken = default)
{
  if (request is null || string.IsNullOrWhiteSpace(request.Email) || string.IsNullOrWhiteSpace(request.Password))
  {
    return new Error("Invalid Credentials");
  }

  /*
  ... Other Code
  */

  var getResult = await this.userRepository.GetAsync(request.Email, cancellationToken);

  if (getResult.IsSuccess)
  {
    return new Conflict("User already exists");
  }

  /*
  ... Other Code
  */

  var createResult = await this.userRepository.CreateAsync(user, cancellationToken: cancellationToken);
  if (!createResult.IsSuccess)
  {
    return new Error("Could not create user");
  }

  return user;
}
```

### Unwrapping a OneOf Instance
Unwrapping is the preffered way.

```csharp
var result = await this.userManager.CreateAsync(request, cancelationToken);
var actionResult = result.Unwrap(
    user => this.Ok(user),
    conflict => this.Conflict(),
    error => this.BadRequest(error.Message)
);
return actionResult;
```

### Checking the result
Sometimes though you need to check what result it is
```csharp
public async Task<OneOf<Ok, NotFound, Conflict, Error>> ChangePasswordAsync(ChangeUserPasswordRequest request, CancellationToken cancellationToken)
{
   /*
  ... Other Code
  */

  var getResult = await this.userRepository.GetAsync(request.Email, cancellationToken);

  if (!getResult.Is<User>())
  {
    // do stuff
  }

  /*
  ... Other Code
  */

  var updateResult = await this.userRepository.UpdateAsync(user, cancellationToken);

  if (!updateResult.Is<Ok>())
  {
    // do stuff
  }

  // do stuff
}
```

## License
This project is licensed under the MIT License.

