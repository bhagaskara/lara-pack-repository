# Laravel Repository Concept (lara-pack/repository)

A minimal structured repository pattern concept package for Laravel, providing a flexible `MasterDataRepository` class to handle common CRUD actions, complex `where` clauses, and dynamic queries for your models.

## Installation

You can install the package via composer:

```bash
composer require lara-pack/repository
```

_(Note: Ensure your package repository is configured locally or the package is published on Packagist)_

## Usage

Create a new repository class and extend `LaraPack\Repository\MasterDataRepository`.

Implement the `className()` method which should return the eloquent `Model` class name this repository handles.

### 1. Creating a Repository

```php
namespace App\Repositories;

use LaraPack\Repository\MasterDataRepository;
use App\Models\User;

class UserRepository extends MasterDataRepository
{
    protected static function className(): string
    {
        return User::class;
    }
}
```

### 2. Available Methods

Here are the various methods you can use out of the box with `MasterDataRepository`.

#### `all()`

Retrieves all records.

```php
$users = UserRepository::all();
```

#### `find($id)`

Finds a specific record by its primary key.

```php
$user = UserRepository::find(1);
```

#### `create($data)`

Creates a new record.

```php
$user = UserRepository::create([
    'name' => 'John Doe',
    'email' => 'john@example.com',
    'password' => bcrypt('password')
]);
```

#### `update($id, $data)`

Updates a record by its primary key.

```php
$status = UserRepository::update(1, ['name' => 'Jane Doe']);
```

#### `delete($id)`

Deletes a record (or soft deletes if the model uses the `SoftDeletes` trait).

```php
$status = UserRepository::delete(1);
```

#### `forceDelete($id)`

Permanently deletes a record bypassing soft deletes.

```php
$status = UserRepository::forceDelete(1);
```

### 3. Dynamic Filtering (`getBy`, `findBy`, `countBy`, `updateBy`, `deleteBy`, `forceDeleteBy`)

The true power of this repository package is its `whereClause` processing. You can dynamically pass arrays of conditions.

#### Structure of Where Clause

Each clause is an array structure that follows this format:

- Basic condition: `['column', 'value']` (operator defaults to `=`)
- With specified operator: `['column', 'operator', 'value']`
- With `IN` operator: `['column', ['value1', 'value2']]`
- With `OR` conjunction: `['column', 'operator', 'value', 'OR']`

#### Retrieve a single record (`findBy`)

```php
// Find the first user where name = 'John'
$user = UserRepository::findBy([
    ['name', '=', 'John']
]);

// You can request a "lock for update" on the row by passing true as the 2nd argument:
$lockedUser = UserRepository::findBy([['id', 1]], true);
```

#### Retrieve multiple records (`getBy`)

```php
// Getting users where status is active AND role IN (admin, manager)
// Order by created_at DESC
$users = UserRepository::getBy(
    [
        ['status', 'active'], // omitted operator defaults to '='
        ['role', ['admin', 'manager']], // using an array as value defaults to 'IN'
    ],
    [
        ['created_at', 'DESC'] // Order by clause
    ]
);

// Advanced conditions (Using OR)
$users = UserRepository::getBy([
    ['role', 'admin'],
    ['status', '=', 'banned', 'OR'] // Will map to OR WHERE status = 'banned'
]);
```

#### Count records (`count()`, `countBy()`)

```php
$totalUsers = UserRepository::count();

$activeUsersCount = UserRepository::countBy([
    ['status', 'active']
]);
```

#### Update or Delete based on conditions

```php
// Update all users where status = 'pending'
UserRepository::updateBy([['status', 'pending']], ['status' => 'active']);

// Delete all users where status = 'banned'
UserRepository::deleteBy([['status', 'banned']]);

// Force delete users based on condition
UserRepository::forceDeleteBy([['status', 'banned']]);
```

## Contributing

Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

## License

[MIT](https://choosealicense.com/licenses/mit/)
