# CSV Context Manager Assignment

This assignment focuses on creating context managers for handling CSV files using two different approaches: a class-based implementation and a decorator-based implementation using generators.

## Concepts Covered

1. Context Managers
2. Generators
3. CSV file handling
4. Named Tuples
5. Lazy Iteration

## Assignment 1: Class-based Context Manager

The goal is to create a context manager using a class that implements the context manager protocol. This context manager should:

- Read CSV files
- Use the first row as field names
- Return data as named tuples
- Provide lazy iteration

Here's the implementation:

```python
import csv
from collections import namedtuple

class CSVContextManager:
    def __init__(self, filename, delimiter=','):
        self.filename = filename  # CSV file path
        self.delimiter = delimiter  # Delimiter for CSV (default is ',')
        self.file = None  # File handle
        self.reader = None  # CSV reader
        self.RowType = None  # Named tuple type

    def __enter__(self):
        # Open the file and create a CSV reader
        self.file = open(self.filename, 'r')
        self.reader = csv.reader(self.file, delimiter=self.delimiter)
        headers = next(self.reader)  # Read the first row as headers
        # Clean header names for valid identifiers
        clean_headers = [h.strip().replace(' ', '_').replace('-', '_') for h in headers]
        self.RowType = namedtuple('Row', clean_headers)  # Create named tuple type
        return self  # Return the context manager instance

    def __exit__(self, exc_type, exc_value, traceback):
        # Close the file when exiting the context
        if self.file:
            self.file.close()
        return False  # Propagate exceptions if any

    def __iter__(self):
        return self  # Return iterator

    def __next__(self):
        try:
            row = next(self.reader)  # Read the next row from the CSV
            return self.RowType(*row)  # Return a named tuple for the row
        except StopIteration:
            raise StopIteration()  # Handle end of iteration
```

## Assignment 2: Decorator-based Context Manager

The second part of the assignment involves recreating the functionality of Assignment 1 using a generator function and the `contextlib.contextmanager` decorator. Here's the implementation:

```python
import csv
from collections import namedtuple
from contextlib import contextmanager

@contextmanager
def csv_context_manager_decorator(filename, delimiter=','):
    file = None  # File handle
    try:
        # Open the file and create a CSV reader
        file = open(filename, 'r')
        reader = csv.reader(file, delimiter=delimiter)
        headers = next(reader)  # Read the first row as headers
        # Clean header names for valid identifiers
        clean_headers = [h.strip().replace(' ', '_').replace('-', '_') for h in headers]
        RowType = namedtuple('Row', clean_headers)  # Create named tuple type
        yield (RowType(*row) for row in reader)  # Yield a generator for lazy iteration
    finally:
        if file:
            file.close()  # Ensure the file is closed
```

## Usage

Both context managers can be used similarly:

```python
# Class-based
with CSVContextManager('nyc_parking_tickets_extract.csv', delimiter=',') as csv_context:
    for row in csv_context:
        print(row)  # Process each row

# Decorator-based
with csv_context_manager_decorator('cars.csv', delimiter=',') as csv_data:
    for row in csv_data:
        print(row)  # Process each row

# Another example with personal_info.csv
with csv_context_manager_decorator('personal_info.csv', delimiter=',') as personal_data:
    for row in personal_data:
        print(row)  # Process each row
```

## Testing

The assignment includes test functions to verify the functionality of both implementations:

```python
def test_csv_context_manager_class():
    csv_file = 'nyc_parking_tickets_extract.csv'
    
    with CSVContextManager(csv_file) as rows:
        for row in rows:
            assert isinstance(row, tuple)  # Check if it's a named tuple
            # Additional assertions can be added based on expected content

def test_csv_context_manager_decorator():
    csv_file = 'cars.csv'
    
    with csv_context_manager_decorator(csv_file,delimiter=';') as rows:
        for row in rows:
            assert isinstance(row, tuple)  # Check if it's a named tuple
            # Additional assertions can be added based on expected content

# Run tests
test_csv_context_manager_class()
test_csv_context_manager_decorator()
```


