# Results

`ChrisMavrommatis.Results` is a .NET library designed to provide utility types for handling results and union types in a clean and expressive manner.

## Installation

Install the package via NuGet Package Manager
```bash
Install-Package ChrisMavrommatis.Results
```

Or via .NET CLI:
```bash
dotnet add package ChrisMavrommatis.Results
```

## Usage
The library includes two main result types:

- `Result<TValue>`
- `OneOf<...>`

# Result
The `Result<TValue>` class represents the outcome of an operation, encapsulating both success and failure states while containing the successful result's object.

Creating a Successful Result
```csharp
var successResult = Result<int>.Successful(42);
Console.WriteLine(successResult.Success); // True
Console.WriteLine(successResult.Value);   // 42
```

Creating a Failed Result
```csharp
var failedResult = Result<int>.Failed("An error occurred");
Console.WriteLine(failedResult.Success);  // False
Console.WriteLine(failedResult.Message);  // "An error occurred"
```

## Void Result
The `Result` class represents the outcome of an operation without containing an object, useful for operations that don't return a value.

## OneOf
The `OneOf<...>` struct represents a value that can be one of several possible types.

Supported OneOf types:

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

    // Other code

    var getResult = await this.userRepository.GetAsync(request.Email, cancellationToken);

    if (getResult.IsSuccess)
    {
        return new Conflict("User already exists");
    }

    // Other code

    var createResult = await this.userRepository.CreateAsync(user, cancellationToken: cancellationToken);
    if (!createResult.IsSuccess)
    {
        return new Error("Could not create user");
    }

    return user;
}
```

### Unwrapping a OneOf Instance
Unwrapping is the preferred way to handle OneOf instances:
```csharp
var result = await this.userManager.CreateAsync(request, cancellationToken);
var actionResult = result.Unwrap(
    user => this.Ok(user),
    conflict => this.Conflict(),
    error => this.BadRequest(error.Message)
);
return actionResult;
```

### Checking the Result
Sometimes you need to check what type of result it is:
```csharp
public async Task<OneOf<Ok, NotFound, Conflict, Error>> ChangePasswordAsync(ChangeUserPasswordRequest request, CancellationToken cancellationToken)
{
    // Other code

    var getResult = await this.userRepository.GetAsync(request.Email, cancellationToken);

    if (!getResult.Is<User>())
    {
        // Handle result
    }

    // Other code

    var updateResult = await this.userRepository.UpdateAsync(user, cancellationToken);

    if (!updateResult.Is<Ok>())
    {
        // Handle result
    }

    // Other code
}
```

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

