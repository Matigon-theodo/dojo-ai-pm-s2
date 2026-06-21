---
name: cosmic-tracing-php
description: PHP-specific supplement for COSMIC tracing (Zend Framework / legacy PHP)
disable-model-invocation: false
---

# PHP-Specific Tracing Supplement

This supplement extends `cosmic-tracing` with PHP-specific source reading and dependency resolution. Loaded alongside the main reference in `cosmic-tracer` agents working on PHP codebases.

## How to Read Source

- Use `php bin/extract-php-function.php <file> <function_name>` to read a function. Never use Read for this. Pass the bare function name (e.g. `removeorderAction`).
- If the extract command fails with "File not found", you are in the wrong directory — run from the project directory where `tracer.yml` is.
- To find a file, use `find` on the `code_path` directory (from `tracer.yml`), e.g. `find <code_path> -name "ClassName.php"`. **Never guess file paths** — always verify with `find`.
- Follow `require_once` / `include` to find function definitions.
- For class methods, resolve inheritance to the file where the method is actually defined.
- File scope (like `login.php`) = all code outside function/class definitions.
- `redirect()` calls `exit` internally, but code paths before the redirect still matter — read the full function.

## PHP Dependency Resolution

### Zend Framework naming conventions

- Class `Module_Model_ClassName` → file `private/modules/module/models/ClassName.php`
- Class `Module_Form_ClassName` → file `private/modules/module/forms/ClassName.php`
- Controller `Module_BackController` → file `private/modules/module/controllers/BackController.php`
- When in doubt, use `find` to locate the file.

### What IS a dependency (PHP-specific examples)

- Direct calls: `db_query(...)` → dep on `func:www/lib/db.php:db_query`
- Method calls: `$result->fetch()` → dep on `func:www/lib/db.php:SQLiteResult.fetch`
- Callbacks: `ob_start('_compta_transform_output')` → dep on the named function
- Included files: `require_once 'header.php'` → dep on `func:www/header.php`

### What is NOT a dependency (PHP-specific)

- PHP builtins: `echo`, `isset`, `empty`, `array_*`, `str_*`, `count`, `trim`, `intval`, etc.
- PHP standard library: `session_start`, `header`, `ob_start`, `exit`, `die`, `date`, `time`, `preg_*`, `json_*`
- PDO methods on `$db_pdo`
- Zend framework methods: `fetchRow`, `fetchAll`, `fetchOne`, `insert`, `update`, `delete`, `getAdapter`, `quoteInto`, `save`, `find`, `_getCols`, `createRow`, `toArray` on Zend_Db_Table/Row objects
- `__callStatic` magic methods used as constants (e.g. `StaticId::CHECKGAMME_LISTE_EXCLUSION__STR()`) — these are lookup tables, not functions
- Doctrine EntityManager methods: `persist`, `flush`, `find`, `getRepository`
- Static constants and class constants: `Product_Model_Product::PRODUCT_REDO`, `Order_Model_Order::STATUS_FACTORY`
