# Welcome to Test

For full document

## Commands

```terraform title=example.tf"
module "project_api_management" {
  source         = "terraform-google-modules/project-factory/google//modules/project_services"
  version        = "~> 14.2"
  project_id     = var.project_id
  activate_apis  = [
    "artifactregistry.googleapis.com",
    "compute.googleapis.com",
    "dns.googleapis.com",
    "secretmanager.googleapis.com",
    "workflows.googleapis.com",
    # ...
  ]
}
```

```tf
module "project_api_management" {
  source         = "terraform-google-modules/project-factory/google//modules/project_services"
  version        = "~> 14.2"
  project_id     = var.project_id
  activate_apis  = [
    "artifactregistry.googleapis.com",
    "compute.googleapis.com",
    "dns.googleapis.com",
    "secretmanager.googleapis.com",
    "workflows.googleapis.com",
    # ...
  ]
}
```

``` terraform
module "project_api_management" {
  source         = "terraform-google-modules/project-factory/google//modules/project_services"
  version        = "~> 14.2"
  project_id     = var.project_id
  activate_apis  = [
    "artifactregistry.googleapis.com",
    "compute.googleapis.com",
    "dns.googleapis.com",
    "secretmanager.googleapis.com",
    "workflows.googleapis.com",
    # ...
  ]
}
```



```
# Function to add two numbers
def add_two_numbers(num1, num2):
    return num1 + num2

# Example usage
result = add_two_numbers(5, 3)
print('The sum is:', result)
```


```py title="add_numbers.py" 
# Function to add two numbers
def add_two_numbers(num1, num2):
    return num1 + num2

# Example usage
result = add_two_numbers(5, 3)
print('The sum is:', result)
```

```py title="add_numbers.py" linenums="1"
# Function to add two numbers
def add_two_numbers(num1, num2):
    return num1 + num2

# Example usage
result = add_two_numbers(5, 3)
print('The sum is:', result)
```

```js title="code-examples.md" linenums="1" hl_lines="2-4"
// Function to concatenate two strings
function concatenateStrings(str1, str2) {
  return str1 + str2;
}

// Example usage
const result = concatenateStrings("Hello, ", "World!");
console.log("The concatenated string is:", result);
```

### Code Blocks in Content Tabs

=== "Python"

    ```py
    def main():
        print("Hello world!")

    if __name__ == "__main__":
        main()
    ```

=== "JavaScript"

    ```js
    function main() {
        console.log("Hello world!");
    }

    main();
    ```