

# Python for DevOps & Cloud Engineering
## A Comprehensive Learning Book: Beginner to Advanced

---

> **Who this book is for:** Cloud & DevOps Engineering students who want to use Python as a professional tool — writing automation scripts, managing cloud infrastructure, building CLI tools, and shipping production-grade code.
>
> **How to use this book:** Read each chapter in order. Every concept builds on the last. Don't skip the tasks — they are where real learning happens.

---

# Introduction: Why Python Is the DevOps Engineer's Best Friend

Before we write a single line of code, let's talk about why you're here.

Imagine you're a new engineer at a company running 200 servers across three cloud providers. Every Monday morning, your team needs to know which servers are running, which ones cost too much, and whether any services are down. Doing this manually would take hours. A Python script can do it in seconds.

That is the power of Python for DevOps and Cloud Engineering. It is not about building apps or websites (though Python does that too). It is about **automation, visibility, and control** over infrastructure at scale.

## What Makes Python Perfect for This Work?

**It reads like English.** Python's syntax is deliberately close to plain language. You can often read Python code and understand what it does even before you learn the language. This means you spend less time fighting syntax and more time solving problems.

**It has a library for everything.** Need to talk to AWS? There's `boto3`. Need to make HTTP requests? There's `requests`. Need to parse YAML config files? There's `pyyaml`. The Python ecosystem is enormous, and the DevOps world has embraced it completely.

**It runs everywhere.** Python works on Linux, macOS, and Windows. All major cloud platforms (AWS, GCP, Azure) provide official Python SDKs. Every CI/CD tool you'll encounter supports Python scripts.

**It scales from a 5-line script to a 50,000-line codebase.** You can write a quick script in an afternoon, and that same language will serve you when you need to build a proper internal platform tool.

## What You Will Learn in This Book

This book is structured into 15 chapters, each covering a core Python skill that a professional DevOps or Cloud Engineer uses regularly:

1. **Python 3 Fundamentals** — the foundation everything else builds on
2. **Object-Oriented Python** — structuring your code for reuse and clarity
3. **File I/O** — reading and writing configuration files in every format
4. **Error Handling** — writing code that doesn't explode in production
5. **Working with the OS** — navigating files, paths, and the system itself
6. **Subprocess and Shell Integration** — running shell commands from Python
7. **HTTP Clients** — talking to APIs and web services
8. **Environment and Configuration** — managing secrets and settings safely
9. **Virtual Environments** — keeping your projects isolated and reproducible
10. **Package Management** — publishing and distributing your own tools
11. **CLI Development** — building command-line tools that feel professional
12. **Cloud SDKs** — controlling AWS, GCP, and Azure from Python
13. **Type Hints, Linting, and Formatting** — writing clean, maintainable code
14. **Testing** — making sure your code works before it hits production
15. **Concurrency** — making your scripts fast with parallel execution

Each chapter ends with a practical task that mirrors a real job scenario. By the end of this book, you will have a portfolio of working scripts that you could use in a real DevOps role.

## A Note on How to Read This Book

Every command, every line of code, every configuration block in this book is explained. Nothing is left to assumption. If you see code you don't understand, keep reading — the explanation follows immediately.

Let's begin.

---

# Chapter 1: Python 3 Fundamentals
## Types, Data Structures, Control Flow, and Functions

---

## Starting from Zero: What Is Python, Really?

Think of Python as a very precise way of giving instructions to your computer. When you speak to a friend, you use natural language — ambiguous, contextual, forgiving. When you write Python, you use a formal language — exact, unambiguous, and literal.

Python is an **interpreted** language, which means your computer reads and executes your Python code line by line, in real time. There's no separate "compilation" step the way there is with C or Go. You write code, you run it, you see results.

Let's verify Python is installed on your system:

```bash
python3 --version
```

You should see something like `Python 3.11.4`. Any version 3.8 or higher works for everything in this book.

---

## Variables: Storing Information

A variable is like a labelled box. You put something in the box, give the box a name, and then you can refer to what's inside by using that name.

```python
server_name = "web-prod-01"
port = 8080
is_running = True
```

Here:
- `server_name` is a variable containing the text `"web-prod-01"`
- `port` is a variable containing the number `8080`
- `is_running` is a variable containing the boolean value `True`

Python figures out what **type** each variable is automatically — this is called **dynamic typing**. You don't need to declare the type yourself (though you can hint at it, which we'll cover in Chapter 13).

---

## Python's Core Data Types

### Strings — Text

Strings are sequences of characters, always wrapped in quotes (single `'` or double `"` — both work identically).

```python
hostname = "prod-server-01"
region = 'us-east-1'

# f-strings let you embed variables directly in text (Python 3.6+)
message = f"Server {hostname} is in region {region}"
print(message)
# Output: Server prod-server-01 is in region us-east-1
```

The `f` before the opening quote turns it into an **f-string** (formatted string literal). Anything inside `{}` gets evaluated and inserted.

Strings are **immutable** — once created, you can't change individual characters. But you can create new strings from existing ones:

```python
path = "/var/log/nginx"
filename = path.split("/")[-1]  # Split on "/" and take the last part
# filename = "nginx"

upper_hostname = hostname.upper()
# upper_hostname = "PROD-SERVER-01"
```

### Integers and Floats — Numbers

```python
cpu_count = 4          # int — whole number
memory_gb = 15.5       # float — decimal number
max_connections = 1_000_000  # underscores make large numbers readable

# Arithmetic
available_memory = memory_gb * 0.8  # 12.4
requests_per_server = max_connections // cpu_count  # 250000 (// = integer division)
remainder = max_connections % cpu_count             # 0 (% = modulo/remainder)
```

### Booleans — True or False

```python
is_healthy = True
is_down = False

# Booleans come from comparisons
port_is_valid = port > 0 and port < 65535  # True
is_prod = server_name.startswith("prod")   # True
```

The keywords `and`, `or`, and `not` combine boolean values:
- `A and B` — True only if both A and B are True
- `A or B` — True if at least one of A or B is True
- `not A` — flips True to False and False to True

### None — The Absence of a Value

```python
response = None  # We haven't fetched it yet

# None is Python's way of saying "no value here"
# It's different from 0, False, or ""
```

---

## Data Structures: Organising Multiple Values

### Lists — Ordered Collections

A list is like a numbered queue. Items stay in order, and you can have duplicates.

```python
servers = ["web-01", "web-02", "db-01", "cache-01"]
ports = [80, 443, 8080, 8443]

# Access by index (counting starts at 0)
first_server = servers[0]   # "web-01"
last_server = servers[-1]   # "cache-01" (negative indexes count from the end)

# Slicing — grab a range of items
web_servers = servers[0:2]  # ["web-01", "web-02"] (from index 0 up to but not including 2)

# Modifying lists
servers.append("db-02")     # Add to end
servers.insert(0, "lb-01")  # Insert at position 0
servers.remove("cache-01")  # Remove by value
popped = servers.pop()      # Remove and return the last item

# Length
server_count = len(servers)

# Check membership
if "web-01" in servers:
    print("web-01 is in the list")
```

### Dictionaries — Key-Value Pairs

A dictionary is like a real-world dictionary: you look something up by its key (the word) and get back a value (the definition). Keys must be unique.

```python
server = {
    "name": "web-prod-01",
    "ip": "10.0.1.100",
    "region": "us-east-1",
    "port": 443,
    "healthy": True
}

# Access by key
print(server["name"])       # "web-prod-01"
print(server.get("port"))   # 443 (use .get() to avoid errors if key doesn't exist)
print(server.get("tags", {}))  # Returns {} if "tags" doesn't exist

# Adding and updating
server["tags"] = {"env": "prod", "team": "platform"}
server["port"] = 8443  # Update existing key

# Removing
del server["healthy"]         # Removes the key entirely
removed = server.pop("ip")    # Removes and returns the value

# Iterating
for key, value in server.items():
    print(f"{key}: {value}")

# Check if key exists
if "name" in server:
    print("Name field present")
```

### Tuples — Immutable Lists

Tuples are like lists, but you can't modify them after creation. Use them when the data should stay fixed.

```python
coordinates = (40.7128, -74.0060)  # latitude, longitude — shouldn't change
rgb_color = (255, 128, 0)

# Unpack a tuple into variables
lat, lon = coordinates
print(f"Lat: {lat}, Lon: {lon}")

# Tuples are often used for multiple return values from functions
def get_connection_info():
    return ("db.prod.internal", 5432)  # host, port

host, port = get_connection_info()
```

### Sets — Unique Collections

A set stores only unique values and has no guaranteed order. Perfect for membership testing and removing duplicates.

```python
active_ips = {"10.0.1.1", "10.0.1.2", "10.0.1.3"}
flagged_ips = {"10.0.1.2", "10.0.1.5"}

# Set operations
both_active_and_flagged = active_ips & flagged_ips    # intersection: {"10.0.1.2"}
either_set = active_ips | flagged_ips                 # union: all unique IPs
only_active = active_ips - flagged_ips                # difference: {"10.0.1.1", "10.0.1.3"}

# Remove duplicates from a list
servers_with_dupes = ["web-01", "web-02", "web-01", "db-01"]
unique_servers = list(set(servers_with_dupes))
```

---

## Control Flow: Making Decisions and Repeating Actions

### if / elif / else

```python
cpu_usage = 87  # percent

if cpu_usage >= 90:
    print("CRITICAL: CPU usage is critical")
elif cpu_usage >= 75:
    print("WARNING: CPU usage is high")
elif cpu_usage >= 50:
    print("INFO: CPU usage is moderate")
else:
    print("OK: CPU usage is normal")

# Output: WARNING: CPU usage is high
```

**Indentation is mandatory in Python.** The code inside each `if`/`elif`/`else` block must be indented consistently (4 spaces is the convention). Python uses indentation to determine block structure — there are no curly braces.

### for Loops — Iterating Over Collections

```python
servers = ["web-01", "web-02", "db-01"]

# Loop over a list
for server in servers:
    print(f"Checking server: {server}")

# Loop with an index
for index, server in enumerate(servers):
    print(f"{index + 1}. {server}")
# Output:
# 1. web-01
# 2. web-02
# 3. db-01

# Loop over a range of numbers
for attempt in range(3):  # 0, 1, 2
    print(f"Attempt {attempt + 1}")

# Loop over a dictionary
config = {"host": "localhost", "port": 5432, "db": "mydb"}
for key, value in config.items():
    print(f"{key} = {value}")
```

### while Loops — Repeating Until a Condition is Met

```python
retries = 0
max_retries = 3
connected = False

while not connected and retries < max_retries:
    print(f"Attempting connection (try {retries + 1})...")
    # Simulate a connection attempt (in real code, this would actually connect)
    retries += 1
    if retries == 2:  # Pretend we succeed on the 2nd try
        connected = True

if connected:
    print("Connected successfully!")
else:
    print("Failed to connect after 3 attempts")
```

### break and continue — Fine-Tuning Loops

```python
servers = ["web-01", "web-02", "MAINTENANCE", "db-01", "db-02"]

for server in servers:
    if server == "MAINTENANCE":
        print("Hit maintenance marker, stopping")
        break   # Exit the loop immediately
    print(f"Processing: {server}")

# Output:
# Processing: web-01
# Processing: web-02
# Hit maintenance marker, stopping

for server in servers:
    if server == "MAINTENANCE":
        continue  # Skip this iteration, move to next
    print(f"Processing: {server}")

# Output:
# Processing: web-01
# Processing: web-02
# Processing: db-01
# Processing: db-02
```

### List Comprehensions — Compact List Building

List comprehensions are a Pythonic way to build a new list by transforming or filtering an existing one. They're common in real Python code.

```python
servers = ["web-01", "web-02", "db-01", "cache-01"]

# Regular loop
web_servers = []
for s in servers:
    if s.startswith("web"):
        web_servers.append(s)

# Same thing as a list comprehension
web_servers = [s for s in servers if s.startswith("web")]
# Result: ["web-01", "web-02"]

# Transform values
upper_servers = [s.upper() for s in servers]
# Result: ["WEB-01", "WEB-02", "DB-01", "CACHE-01"]

# Dictionary comprehension
server_status = {s: "unknown" for s in servers}
# Result: {"web-01": "unknown", "web-02": "unknown", ...}
```

---

## Functions: Reusable Blocks of Logic

A function is a named, reusable chunk of code. You define it once, and call it whenever you need it.

```python
def check_server_health(hostname, port=80, timeout=5):
    """
    Check if a server is responding.
    
    Args:
        hostname: The server's hostname or IP address.
        port: The port to check (defaults to 80).
        timeout: How many seconds to wait (defaults to 5).
    
    Returns:
        A dictionary with 'healthy' (bool) and 'message' (str).
    """
    # In a real script, we'd actually try to connect here
    # For now, let's simulate it
    if not hostname:
        return {"healthy": False, "message": "No hostname provided"}
    
    return {
        "healthy": True,
        "message": f"Successfully reached {hostname}:{port}"
    }

# Calling with positional arguments
result = check_server_health("web-prod-01")
print(result)  # {'healthy': True, 'message': 'Successfully reached web-prod-01:80'}

# Calling with keyword arguments (order doesn't matter when using keyword args)
result = check_server_health(hostname="db-prod-01", timeout=10, port=5432)
```

Breaking this down:
- `def` declares a function
- `hostname, port=80, timeout=5` are the **parameters** — `port` and `timeout` have **default values**, so they're optional when calling
- The triple-quoted string at the top is a **docstring** — documentation built into the function
- `return` sends a value back to whoever called the function

### *args and **kwargs — Flexible Arguments

Sometimes you don't know in advance how many arguments a function will receive:

```python
def log_event(*args, **kwargs):
    """
    *args collects any number of positional arguments as a tuple.
    **kwargs collects any number of keyword arguments as a dictionary.
    """
    print("Positional args:", args)
    print("Keyword args:", kwargs)

log_event("server-down", "critical", server="web-01", timestamp="2024-01-15T10:00:00")
# Positional args: ('server-down', 'critical')
# Keyword args: {'server': 'web-01', 'timestamp': '2024-01-15T10:00:00'}
```

### Lambda Functions — Anonymous One-Liners

Lambdas are tiny, unnamed functions useful for short operations:

```python
servers = [
    {"name": "web-01", "cpu": 45},
    {"name": "db-01", "cpu": 89},
    {"name": "cache-01", "cpu": 12},
]

# Sort by CPU usage using a lambda
sorted_by_cpu = sorted(servers, key=lambda s: s["cpu"], reverse=True)
# Result: db-01 (89), web-01 (45), cache-01 (12)
```

---

## Common Beginner Mistakes

**Mistake 1: Mutable default arguments**

```python
# WRONG — the list is shared across all calls!
def add_server(name, server_list=[]):
    server_list.append(name)
    return server_list

print(add_server("web-01"))  # ['web-01']
print(add_server("web-02"))  # ['web-01', 'web-02'] — unexpected!

# CORRECT — use None as default, create fresh list inside
def add_server(name, server_list=None):
    if server_list is None:
        server_list = []
    server_list.append(name)
    return server_list
```

**Mistake 2: Modifying a list while iterating over it**

```python
servers = ["web-01", "web-02", "DEAD", "db-01"]

# WRONG — skips items because the list changes mid-loop
for server in servers:
    if server == "DEAD":
        servers.remove(server)

# CORRECT — iterate over a copy
for server in servers[:]:  # servers[:] creates a copy
    if server == "DEAD":
        servers.remove(server)

# ALSO CORRECT — build a new list
servers = [s for s in servers if s != "DEAD"]
```

**Mistake 3: == vs is**

```python
x = None

# WRONG for None comparison
if x == None:  # Works but not Pythonic
    pass

# CORRECT — use 'is' for None, True, False
if x is None:
    pass
```

---

## How This Works in the Real World

Every automation script starts here. In practice, a DevOps engineer might write a script like this to check a list of servers:

```python
def get_server_inventory():
    """Returns a list of servers to monitor."""
    return [
        {"name": "web-prod-01", "ip": "10.0.1.10", "port": 443},
        {"name": "web-prod-02", "ip": "10.0.1.11", "port": 443},
        {"name": "db-prod-01",  "ip": "10.0.2.10", "port": 5432},
        {"name": "cache-prod",  "ip": "10.0.3.10", "port": 6379},
    ]

def summarise_inventory(servers):
    """Print a summary of the server inventory."""
    total = len(servers)
    unique_ports = set(s["port"] for s in servers)
    
    print(f"Total servers: {total}")
    print(f"Unique ports in use: {sorted(unique_ports)}")
    
    # Group by port
    by_port = {}
    for server in servers:
        port = server["port"]
        if port not in by_port:
            by_port[port] = []
        by_port[port].append(server["name"])
    
    for port, names in by_port.items():
        print(f"  Port {port}: {', '.join(names)}")

inventory = get_server_inventory()
summarise_inventory(inventory)
```

This simple script demonstrates variables, lists, dictionaries, for loops, functions, and set operations — all working together in a realistic context.

---

## Chapter 1 Summary

- Python variables are dynamically typed — the type is inferred automatically
- Core types: `str`, `int`, `float`, `bool`, `None`
- Core data structures: `list` (ordered, mutable), `dict` (key-value), `tuple` (ordered, immutable), `set` (unique values)
- Control flow uses `if/elif/else`, `for`, and `while`
- Functions are defined with `def`, support default arguments, and can return any value
- Indentation (4 spaces) defines code blocks — it is not optional
- List comprehensions are a Pythonic way to transform and filter lists

---

# Chapter 2: Object-Oriented Python
## Classes, Inheritance, and Dataclasses

---

## What Is Object-Oriented Programming?

Before the jargon, let's use an analogy.

Imagine you're managing a fleet of servers. Every server has properties (a name, an IP address, a region, a status) and things it can do (be checked, be restarted, be decommissioned). 

Now imagine you have 50 of these servers. Without some structure, your code would be a sprawl of variables and functions with no clear organisation. You'd have `server1_name`, `server1_ip`, `server2_name`, `server2_ip`... and then `check_server1()`, `check_server2()`...

Object-Oriented Programming (OOP) solves this by letting you create a **blueprint** (a **class**) that describes what a server is and what it can do. Then you create individual **instances** of that blueprint (called **objects**), each with their own data.

The class is the blueprint. The object is the building.

---

## Defining a Class

```python
class Server:
    """Represents a single server in our infrastructure."""
    
    # Class variable — shared by ALL instances
    supported_regions = ["us-east-1", "us-west-2", "eu-west-1"]
    
    def __init__(self, name, ip_address, region, port=443):
        """
        The __init__ method is the constructor — it runs when you create a new Server.
        'self' refers to the specific instance being created.
        """
        self.name = name              # Instance variable — unique to each Server
        self.ip_address = ip_address
        self.region = region
        self.port = port
        self.is_healthy = None        # Unknown until we check
        self._restart_count = 0       # Convention: _ prefix means "private, internal use"
    
    def check_health(self):
        """Simulate a health check (in real code, you'd make an HTTP request)."""
        # Pretend the server is healthy
        self.is_healthy = True
        return self.is_healthy
    
    def restart(self):
        """Record a restart."""
        self._restart_count += 1
        self.is_healthy = None  # Unknown after restart until we check
        print(f"Server {self.name} restarted (total restarts: {self._restart_count})")
    
    def __repr__(self):
        """
        __repr__ is called when Python needs to show the object as a string,
        such as in print() or the REPL.
        """
        status = "healthy" if self.is_healthy else "unknown"
        return f"Server(name={self.name!r}, region={self.region!r}, status={status})"
    
    def __eq__(self, other):
        """Define what 'equal' means for two Server objects."""
        if not isinstance(other, Server):
            return NotImplemented
        return self.name == other.name and self.ip_address == other.ip_address
```

Now let's use this class:

```python
# Creating instances (objects)
web01 = Server("web-prod-01", "10.0.1.10", "us-east-1")
web02 = Server("web-prod-02", "10.0.1.11", "us-east-1", port=8443)
db01  = Server("db-prod-01",  "10.0.2.10", "us-west-2", port=5432)

# Accessing attributes
print(web01.name)       # "web-prod-01"
print(web01.port)       # 443

# Calling methods
web01.check_health()
print(web01.is_healthy) # True

web01.restart()         # Server web-prod-01 restarted (total restarts: 1)

# Using __repr__
print(web01)  # Server(name='web-prod-01', region='us-east-1', status=unknown)

# Accessing class variables
print(Server.supported_regions)  # Works on the class itself
print(web01.supported_regions)   # Also works on instances
```

---

## Properties: Controlled Attribute Access

Sometimes you want to control how an attribute is read or written. Python's `@property` decorator lets you do this without changing how the calling code looks:

```python
class Server:
    def __init__(self, name, ip_address, region):
        self.name = name
        self.ip_address = ip_address
        self.region = region
        self._status = "unknown"  # Store the actual value with underscore
    
    @property
    def status(self):
        """Getter — called when you read server.status"""
        return self._status
    
    @status.setter
    def status(self, value):
        """Setter — called when you write server.status = 'healthy'"""
        valid_statuses = {"healthy", "unhealthy", "unknown", "maintenance"}
        if value not in valid_statuses:
            raise ValueError(f"Invalid status: {value!r}. Must be one of {valid_statuses}")
        self._status = value

web01 = Server("web-prod-01", "10.0.1.10", "us-east-1")
web01.status = "healthy"   # Calls the setter
print(web01.status)         # "healthy" — calls the getter

web01.status = "broken"     # Raises ValueError: Invalid status: 'broken'
```

---

## Inheritance: Extending a Class

Inheritance lets you create a new class that gets all the functionality of an existing class, plus its own additions. Think of it as a more specific type of the original.

```python
class Server:
    """Base class for all server types."""
    
    def __init__(self, name, ip_address, region):
        self.name = name
        self.ip_address = ip_address
        self.region = region
    
    def describe(self):
        return f"Server: {self.name} at {self.ip_address} in {self.region}"
    
    def health_check_url(self):
        raise NotImplementedError("Subclasses must implement health_check_url()")


class WebServer(Server):
    """A web server — inherits from Server and adds web-specific functionality."""
    
    def __init__(self, name, ip_address, region, ssl_enabled=True):
        # Call the parent class constructor
        super().__init__(name, ip_address, region)
        self.ssl_enabled = ssl_enabled
        self.virtual_hosts = []
    
    def health_check_url(self):
        """Implement the abstract method from Server."""
        protocol = "https" if self.ssl_enabled else "http"
        return f"{protocol}://{self.ip_address}/health"
    
    def add_vhost(self, domain):
        self.virtual_hosts.append(domain)


class DatabaseServer(Server):
    """A database server with its own health check approach."""
    
    def __init__(self, name, ip_address, region, db_engine="postgres", port=5432):
        super().__init__(name, ip_address, region)
        self.db_engine = db_engine
        self.port = port
    
    def health_check_url(self):
        return f"postgresql://{self.ip_address}:{self.port}/health"
    
    def describe(self):
        # Override the parent's describe() method
        base = super().describe()  # Get the parent's output
        return f"{base} [{self.db_engine}:{self.port}]"


# Usage
web = WebServer("web-01", "10.0.1.10", "us-east-1")
web.add_vhost("api.example.com")
print(web.health_check_url())  # https://10.0.1.10/health
print(web.describe())           # Server: web-01 at 10.0.1.10 in us-east-1

db = DatabaseServer("db-01", "10.0.2.10", "us-east-1")
print(db.health_check_url())   # postgresql://10.0.2.10:5432/health
print(db.describe())            # Server: db-01 at 10.0.2.10 in us-east-1 [postgres:5432]

# isinstance() checks — useful in real code
print(isinstance(web, WebServer))   # True
print(isinstance(web, Server))      # True — because WebServer inherits from Server
print(isinstance(db, WebServer))    # False
```

---

## Dataclasses: The Modern Way to Create Simple Classes

Python 3.7 introduced **dataclasses** — a decorator that auto-generates boilerplate code (`__init__`, `__repr__`, `__eq__`) for classes that are primarily data containers.

Instead of writing all that yourself, you just declare the fields:

```python
from dataclasses import dataclass, field
from typing import Optional, List

@dataclass
class ServerConfig:
    """Configuration for a server. A dataclass auto-generates __init__, __repr__, __eq__."""
    
    name: str                              # Required field
    ip_address: str                        # Required field
    region: str                            # Required field
    port: int = 443                        # Optional field with default
    ssl_enabled: bool = True               # Optional field with default
    tags: dict = field(default_factory=dict)     # Mutable defaults use field()
    virtual_hosts: list = field(default_factory=list)
    health_path: str = "/health"
    
    # Post-init validation
    def __post_init__(self):
        if not self.name:
            raise ValueError("Server name cannot be empty")
        if not (0 < self.port < 65536):
            raise ValueError(f"Port {self.port} is out of range")

# Usage — notice how clean this is
config = ServerConfig(
    name="web-prod-01",
    ip_address="10.0.1.10",
    region="us-east-1",
    tags={"env": "prod", "team": "platform"},
)

print(config)
# ServerConfig(name='web-prod-01', ip_address='10.0.1.10', region='us-east-1', port=443, ...)

print(config.name)     # web-prod-01
print(config.port)     # 443 (default)

# Equality is automatically defined based on all fields
config2 = ServerConfig(name="web-prod-01", ip_address="10.0.1.10", region="us-east-1")
print(config == config2)  # True
```

**When to use dataclasses vs regular classes:**
- Use a **dataclass** when the class is mainly a container for data — configuration objects, API response models, inventory records
- Use a **regular class** when the class has complex logic, state that needs careful management, or non-trivial methods

---

## Common Beginner Mistakes

**Mistake 1: Forgetting `self`**

```python
class Counter:
    def __init__(self):
        self.count = 0
    
    def increment():   # WRONG — missing self
        count += 1     # This fails too — count is not defined in this scope
    
    def increment(self):  # CORRECT
        self.count += 1
```

**Mistake 2: Shared mutable class variables**

```python
# WRONG — all instances share the same list!
class ServerGroup:
    servers = []  # Class variable
    
    def add(self, name):
        self.servers.append(name)

g1 = ServerGroup()
g2 = ServerGroup()
g1.add("web-01")
print(g2.servers)  # ["web-01"] — unexpected!

# CORRECT — use instance variables
class ServerGroup:
    def __init__(self):
        self.servers = []  # Instance variable — each instance gets its own list
    
    def add(self, name):
        self.servers.append(name)
```

---

## How This Works in the Real World

In real DevOps tooling, you'll see OOP used to model infrastructure. AWS's `boto3` library itself is built on classes — every AWS service you interact with is an object. When you write infrastructure-as-code tools, dataclasses shine for representing configuration schemas:

```python
from dataclasses import dataclass, field
from typing import List, Optional

@dataclass
class EC2InstanceConfig:
    instance_type: str
    ami_id: str
    region: str
    key_name: Optional[str] = None
    security_group_ids: List[str] = field(default_factory=list)
    subnet_id: Optional[str] = None
    tags: dict = field(default_factory=dict)
    
    def to_boto3_params(self):
        """Convert this config object into the dict format boto3 expects."""
        params = {
            "ImageId": self.ami_id,
            "InstanceType": self.instance_type,
            "MinCount": 1,
            "MaxCount": 1,
        }
        if self.key_name:
            params["KeyName"] = self.key_name
        if self.security_group_ids:
            params["SecurityGroupIds"] = self.security_group_ids
        if self.subnet_id:
            params["SubnetId"] = self.subnet_id
        if self.tags:
            params["TagSpecifications"] = [{
                "ResourceType": "instance",
                "Tags": [{"Key": k, "Value": v} for k, v in self.tags.items()]
            }]
        return params
```

---

## Chapter 2 Summary

- A **class** is a blueprint; an **object** (instance) is a specific thing built from that blueprint
- `__init__` is the constructor — it runs when you create an instance
- `self` refers to the specific instance — always the first parameter of instance methods
- **Inheritance** lets a subclass reuse and extend a parent class; `super()` accesses the parent
- `@property` creates controlled access to attributes
- **Dataclasses** eliminate boilerplate for data-focused classes — use `@dataclass` and declare typed fields
- Use `_ prefix` for internal/private attributes by convention

---

# Chapter 3: File I/O
## Reading, Writing, CSV, JSON, YAML, TOML, and XML

---

## Why File I/O Matters for DevOps

Nearly everything in DevOps involves files. Configuration files, log files, inventory files, Terraform state files, Kubernetes manifests, Ansible playbooks — they're all files. A DevOps engineer who can read, write, and transform files programmatically is dramatically more efficient than one who does it by hand.

In this chapter, we'll cover how Python handles files, and then dive into the specific formats you'll encounter constantly: JSON, YAML, TOML, CSV, and XML.

---

## Basic File Reading and Writing

### Reading a File

```python
# Method 1: Open and read manually (always close the file)
file = open("/etc/hostname", "r")  # "r" = read mode
content = file.read()              # Read the entire file as a string
file.close()                       # MUST close to release the resource

# Method 2: Using 'with' — the Pythonic way (automatically closes the file)
with open("/etc/hostname", "r") as f:
    content = f.read()
# File is automatically closed when the 'with' block exits — even if an error occurs

print(content.strip())  # .strip() removes leading/trailing whitespace (including newlines)
```

The `with` statement is the correct way to work with files. It guarantees the file is closed even if an exception occurs during reading. Always use `with`.

### Reading Line by Line

```python
# Read all lines at once (fine for small files)
with open("/var/log/app.log", "r") as f:
    lines = f.readlines()  # Returns a list of strings, each ending with \n

for line in lines:
    print(line.strip())

# Iterate line by line (memory-efficient for large files)
with open("/var/log/app.log", "r") as f:
    for line in f:  # f is iterable — yields one line at a time
        if "ERROR" in line:
            print(line.strip())
```

### Writing a File

```python
# "w" = write mode (creates file if it doesn't exist, OVERWRITES if it does)
with open("/tmp/servers.txt", "w") as f:
    f.write("web-prod-01\n")
    f.write("web-prod-02\n")
    f.write("db-prod-01\n")

# "a" = append mode (adds to end of file without overwriting)
with open("/tmp/servers.txt", "a") as f:
    f.write("cache-prod-01\n")

# Write multiple lines at once
servers = ["web-01\n", "web-02\n", "db-01\n"]
with open("/tmp/servers.txt", "w") as f:
    f.writelines(servers)  # Note: writelines does NOT add newlines automatically

# Better: use join
servers = ["web-01", "web-02", "db-01"]
with open("/tmp/servers.txt", "w") as f:
    f.write("\n".join(servers) + "\n")  # Join with newlines, add final newline
```

### File Encoding

Always specify encoding when working with text files, especially if they might contain non-ASCII characters:

```python
with open("config.txt", "r", encoding="utf-8") as f:
    content = f.read()
```

UTF-8 is the correct encoding for nearly all modern text files. Specifying it explicitly prevents errors on systems where the default might differ.

---

## JSON — JavaScript Object Notation

JSON is the lingua franca of APIs and modern configuration. It maps almost perfectly to Python's built-in data structures.

```json
{
  "servers": [
    {"name": "web-01", "ip": "10.0.1.10", "healthy": true},
    {"name": "db-01",  "ip": "10.0.2.10", "healthy": false}
  ],
  "last_checked": "2024-01-15T10:00:00Z",
  "check_interval_seconds": 60
}
```

```python
import json

# --- Reading JSON from a file ---
with open("infrastructure.json", "r") as f:
    data = json.load(f)         # Parses JSON directly from a file object
    # OR: content = f.read() then json.loads(content) — loads() takes a string

print(data["servers"][0]["name"])  # "web-01"
print(data["check_interval_seconds"])  # 60

# JSON true/false -> Python True/False
print(data["servers"][0]["healthy"])  # True

# --- Writing JSON to a file ---
report = {
    "generated_at": "2024-01-15T10:05:00Z",
    "total_servers": 2,
    "healthy_count": 1,
    "unhealthy": ["db-01"]
}

with open("report.json", "w") as f:
    json.dump(report, f, indent=2)   # indent=2 makes it human-readable
    # Without indent=2, it writes as a single line

# --- Converting between JSON string and Python object ---
json_string = '{"name": "web-01", "port": 443}'
obj = json.loads(json_string)   # string -> Python dict
print(obj["port"])              # 443

back_to_string = json.dumps(obj, indent=2)  # Python dict -> string
print(back_to_string)
```

**JSON type mapping:**

| JSON Type | Python Type |
|-----------|-------------|
| object `{}` | `dict` |
| array `[]` | `list` |
| string `"..."` | `str` |
| number | `int` or `float` |
| `true` / `false` | `True` / `False` |
| `null` | `None` |

---

## YAML — Yet Another Markup Language

YAML is the dominant format for DevOps configuration — Kubernetes manifests, Ansible playbooks, GitHub Actions workflows, Docker Compose files. It's more readable than JSON for humans.

```yaml
# infrastructure.yaml
servers:
  - name: web-01
    ip: 10.0.1.10
    port: 443
    tags:
      env: prod
      team: platform
  - name: db-01
    ip: 10.0.2.10
    port: 5432
    
monitoring:
  enabled: true
  interval_seconds: 30
  alert_channels:
    - slack
    - pagerduty
```

```python
# Install: pip install pyyaml
import yaml

# --- Reading YAML ---
with open("infrastructure.yaml", "r") as f:
    config = yaml.safe_load(f)   # ALWAYS use safe_load, never just yaml.load()

print(config["servers"][0]["name"])    # "web-01"
print(config["monitoring"]["enabled"]) # True

# --- Writing YAML ---
data = {
    "deployment": {
        "replicas": 3,
        "image": "myapp:v1.2.0",
        "resources": {
            "memory": "512Mi",
            "cpu": "250m"
        }
    }
}

with open("deployment.yaml", "w") as f:
    yaml.dump(data, f, default_flow_style=False, indent=2)
    # default_flow_style=False forces block style (the readable multi-line format)
```

**Why `safe_load` and not `yaml.load`?** Plain `yaml.load()` can execute arbitrary Python code embedded in YAML files. This is a serious security vulnerability. `yaml.safe_load()` only parses data — never use `yaml.load()` without the `Loader=yaml.SafeLoader` argument.

---

## TOML — Tom's Obvious Minimal Language

TOML is gaining popularity for Python project configuration (`pyproject.toml`), Rust packages, and other tools. Python 3.11+ includes TOML support in the standard library.

```toml
# config.toml
[app]
name = "health-checker"
version = "1.0.0"
debug = false

[database]
host = "db.prod.internal"
port = 5432
name = "appdb"

[monitoring]
enabled = true
interval = 30
channels = ["slack", "pagerduty"]
```

```python
# Python 3.11+: built into the standard library as 'tomllib' (read-only)
import tomllib   # Python 3.11+

with open("config.toml", "rb") as f:   # Note: "rb" (read binary) is required for tomllib
    config = tomllib.load(f)

print(config["app"]["name"])          # "health-checker"
print(config["database"]["port"])     # 5432
print(config["monitoring"]["channels"])  # ['slack', 'pagerduty']

# For Python 3.10 and below, or for writing TOML:
# pip install tomli (read) or pip install tomli-w (write)
# pip install tomllib-w  (for writing in 3.11+)
import tomli_w   # pip install tomli-w

with open("output.toml", "wb") as f:  # "wb" (write binary) required
    tomli_w.dump(config, f)
```

---

## CSV — Comma-Separated Values

CSV files are everywhere in operations — exported from monitoring tools, billing reports, server inventories. Python's `csv` module is in the standard library.

```csv
name,ip_address,region,port,healthy
web-prod-01,10.0.1.10,us-east-1,443,true
web-prod-02,10.0.1.11,us-east-1,443,true
db-prod-01,10.0.2.10,us-east-1,5432,false
cache-prod,10.0.3.10,us-west-2,6379,true
```

```python
import csv

# --- Reading CSV as dictionaries (the most useful way) ---
with open("servers.csv", "r", newline="", encoding="utf-8") as f:
    # newline="" is important — the csv module handles newlines itself
    reader = csv.DictReader(f)  # Uses first row as header/field names
    
    for row in reader:
        # Each row is an OrderedDict (or dict in Python 3.8+)
        print(f"{row['name']} ({row['ip_address']}) - healthy: {row['healthy']}")

# Collect all rows into a list
with open("servers.csv", "r", newline="", encoding="utf-8") as f:
    reader = csv.DictReader(f)
    servers = list(reader)

print(servers[0])  # {'name': 'web-prod-01', 'ip_address': '10.0.1.10', ...}

# --- Writing CSV ---
servers = [
    {"name": "web-01", "ip": "10.0.1.10", "region": "us-east-1", "port": 443},
    {"name": "db-01",  "ip": "10.0.2.10", "region": "us-east-1", "port": 5432},
]

with open("output.csv", "w", newline="", encoding="utf-8") as f:
    fieldnames = ["name", "ip", "region", "port"]
    writer = csv.DictWriter(f, fieldnames=fieldnames)
    
    writer.writeheader()           # Writes the header row
    writer.writerows(servers)      # Write all rows at once
    # OR: for row in servers: writer.writerow(row)
```

---

## XML — Extensible Markup Language

XML is older than JSON but still common — AWS API responses, configuration files for Java applications, Ant build files. Python's `xml.etree.ElementTree` is in the standard library.

```xml
<!-- servers.xml -->
<inventory>
    <server name="web-prod-01" region="us-east-1">
        <ip>10.0.1.10</ip>
        <port>443</port>
        <tags>
            <tag key="env" value="prod"/>
            <tag key="team" value="platform"/>
        </tags>
    </server>
    <server name="db-prod-01" region="us-east-1">
        <ip>10.0.2.10</ip>
        <port>5432</port>
    </server>
</inventory>
```

```python
import xml.etree.ElementTree as ET

# --- Parsing XML ---
tree = ET.parse("servers.xml")
root = tree.getroot()   # root is the <inventory> element

print(root.tag)  # "inventory"

for server in root.findall("server"):
    # Attributes are accessed with .get()
    name = server.get("name")
    region = server.get("region")
    
    # Child elements are accessed with .find() or .findall()
    ip = server.find("ip").text       # .text is the text content of an element
    port = server.find("port").text
    
    # Handle optional elements safely
    tags_elem = server.find("tags")
    tags = {}
    if tags_elem is not None:
        for tag in tags_elem.findall("tag"):
            tags[tag.get("key")] = tag.get("value")
    
    print(f"{name} ({ip}:{port}) tags: {tags}")

# --- Building XML ---
root = ET.Element("inventory")    # Create root element

server_elem = ET.SubElement(root, "server")  # Add child element
server_elem.set("name", "web-new-01")        # Set attribute
server_elem.set("region", "us-west-2")

ip_elem = ET.SubElement(server_elem, "ip")
ip_elem.text = "10.0.4.10"                   # Set text content

# Write to file
ET.indent(root, space="    ")  # Pretty-print (Python 3.9+)
tree = ET.ElementTree(root)
tree.write("output.xml", encoding="unicode", xml_declaration=True)
```

---

## Common Beginner Mistakes

**Mistake 1: Not using `with` for file operations**

```python
# WRONG — if an exception occurs before close(), the file stays open
f = open("data.json")
data = json.load(f)
f.close()  # Might not be reached

# CORRECT
with open("data.json") as f:
    data = json.load(f)
```

**Mistake 2: Using `yaml.load()` without a Loader**

```python
# DANGEROUS — can execute arbitrary code in the YAML file
data = yaml.load(content)

# SAFE — always use safe_load
data = yaml.safe_load(content)
```

**Mistake 3: Forgetting `newline=""` in CSV**

```python
# WRONG on Windows — produces extra blank lines
with open("data.csv", "w") as f:
    writer = csv.writer(f)

# CORRECT — always pass newline=""
with open("data.csv", "w", newline="") as f:
    writer = csv.writer(f)
```

---

## How This Works in the Real World

Config loaders are everywhere in real DevOps tooling. Here's a pattern you'll see and write often — reading a YAML/JSON config and merging in environment variable overrides:

```python
import json
import yaml
import os
from pathlib import Path

def load_config(config_path):
    """Load a YAML or JSON config file."""
    path = Path(config_path)
    
    with open(path, "r") as f:
        if path.suffix in (".yaml", ".yml"):
            return yaml.safe_load(f)
        elif path.suffix == ".json":
            return json.load(f)
        else:
            raise ValueError(f"Unsupported config format: {path.suffix}")

def apply_env_overrides(config, prefix="APP_"):
    """
    Override config values from environment variables.
    APP_DATABASE__HOST=newhost overrides config['database']['host']
    Double underscore (__) represents nesting.
    """
    for key, value in os.environ.items():
        if key.startswith(prefix):
            config_key = key[len(prefix):].lower()
            parts = config_key.split("__")
            
            # Navigate to the right level in the config dict
            target = config
            for part in parts[:-1]:
                target = target.setdefault(part, {})
            target[parts[-1]] = value
    
    return config
```

---

## Chapter 3 Summary

- Use `with open(...) as f:` for all file operations — it guarantees proper cleanup
- `"r"` = read, `"w"` = write (overwrites), `"a"` = append; add `b` for binary (`"rb"`, `"wb"`)
- JSON: `json.load(f)` from file, `json.loads(str)` from string; `json.dump()` and `json.dumps()` to write
- YAML: `yaml.safe_load(f)` to read (never `yaml.load()`); `yaml.dump()` to write
- TOML: `tomllib.load(f)` (Python 3.11+) requires binary read mode (`"rb"`)
- CSV: `csv.DictReader` for reading rows as dicts; `csv.DictWriter` for writing; always pass `newline=""`
- XML: `ET.parse()` to load; `.findall()`, `.find()`, `.get()`, `.text` to navigate

---

# Chapter 4: Error Handling
## try/except/else/finally and Custom Exceptions

---

## Why Error Handling Is Not Optional

Imagine deploying an automation script at 3am that checks every server in your fleet for disk space issues. It runs fine for the first 50 servers, then crashes because server 51 has a network timeout. The script dies, leaving the other 150 servers unchecked. You wake up to a full alert storm.

This is what happens without proper error handling. In production code — especially scripts that run unattended — errors are not exceptional. They're expected. Networks time out, files don't exist, APIs return unexpected responses, disk space runs out. Your code must handle these situations gracefully.

---

## The Anatomy of Exception Handling

```python
try:
    # The code that might fail
    result = 10 / 0
except ZeroDivisionError:
    # Runs ONLY if the specific exception occurs
    print("Cannot divide by zero!")
else:
    # Runs ONLY if NO exception occurred
    print(f"Result: {result}")
finally:
    # ALWAYS runs, whether or not an exception occurred
    print("This always runs — use for cleanup")
```

Let's break down each clause:

- **`try`** — the block where you put risky code. Python attempts to run it.
- **`except`** — runs if a specific exception type is raised. You can have multiple `except` blocks.
- **`else`** — runs only if the `try` block completed without raising any exception. Use it for code that should only run on success.
- **`finally`** — runs no matter what. Use it for cleanup (closing files, closing connections, releasing locks).

---

## Python's Exception Hierarchy

Python exceptions are organised in a hierarchy. Knowing this helps you catch exceptions at the right level of specificity:

```
BaseException
 ├── SystemExit           (sys.exit() was called)
 ├── KeyboardInterrupt    (user pressed Ctrl+C)
 └── Exception            (base for all "normal" errors)
      ├── ValueError       (invalid value, e.g. int("hello"))
      ├── TypeError        (wrong type, e.g. "hello" + 5)
      ├── KeyError         (dict key not found)
      ├── IndexError       (list index out of range)
      ├── AttributeError   (object has no such attribute)
      ├── FileNotFoundError (file doesn't exist)
      ├── PermissionError  (no permission to read/write)
      ├── TimeoutError     (operation timed out)
      ├── ConnectionError  (network connection failed)
      │    └── ConnectionRefusedError
      └── OSError          (OS-level error)
```

---

## Catching Exceptions

### Catching a Specific Exception

```python
import json

def load_config(filepath):
    try:
        with open(filepath, "r") as f:
            return json.load(f)
    except FileNotFoundError:
        print(f"Config file not found: {filepath}")
        return {}
    except json.JSONDecodeError as e:
        print(f"Invalid JSON in {filepath}: {e}")
        return {}
```

The `as e` part captures the exception object, giving you access to the error message and other details.

### Catching Multiple Exception Types

```python
def fetch_server_data(server_ip, port):
    import socket
    
    try:
        sock = socket.create_connection((server_ip, port), timeout=5)
        data = sock.recv(1024)
        sock.close()
        return data
    
    except socket.timeout:
        print(f"Connection to {server_ip}:{port} timed out")
        return None
    
    except ConnectionRefusedError:
        print(f"Connection refused by {server_ip}:{port}")
        return None
    
    except OSError as e:
        # Catch broader OS errors (network unreachable, etc.)
        print(f"Network error connecting to {server_ip}:{port}: {e}")
        return None
```

### Catching Multiple Exceptions in One Handler

```python
try:
    process_config_file("config.yaml")
except (FileNotFoundError, PermissionError) as e:
    # Both exceptions handled the same way
    print(f"Cannot read config file: {e}")
```

### The else Clause

```python
def parse_port(port_string):
    """Parse a port string into an integer, with validation."""
    try:
        port = int(port_string)
    except ValueError:
        print(f"'{port_string}' is not a valid integer")
        return None
    else:
        # Only runs if int() succeeded
        if not (1 <= port <= 65535):
            print(f"Port {port} is out of valid range (1-65535)")
            return None
        return port

print(parse_port("443"))     # 443
print(parse_port("hello"))   # None
print(parse_port("99999"))   # None
```

Using `else` separates the "what might fail" code from the "what to do on success" code, making logic clearer.

### The finally Clause

```python
def process_log_file(filepath):
    f = None
    try:
        f = open(filepath, "r")
        return process_lines(f.readlines())
    except FileNotFoundError:
        print(f"Log file not found: {filepath}")
        return []
    finally:
        # This runs EVEN IF an exception occurred and EVEN IF return was called
        if f is not None:
            f.close()
            print("File closed")

# In practice, you should use 'with' instead — it handles this automatically
# But finally is essential for connections, database transactions, lock releases
```

---

## Raising Exceptions

You can raise exceptions yourself when your code encounters an invalid state:

```python
def validate_region(region):
    valid_regions = {"us-east-1", "us-west-2", "eu-west-1", "ap-southeast-1"}
    if region not in valid_regions:
        raise ValueError(f"Invalid region: {region!r}. Must be one of: {valid_regions}")
    return region

def connect_to_aws(region):
    validate_region(region)  # Will raise ValueError if invalid
    # ... continue with connection

# Re-raising an exception after logging it
def run_health_check(server):
    try:
        result = check_server(server)
        return result
    except Exception as e:
        print(f"Health check failed for {server}: {e}")
        raise  # Re-raise the SAME exception — don't lose the original error
```

---

## Creating Custom Exceptions

Custom exceptions let you create a vocabulary of errors specific to your application. This makes error handling at higher levels much cleaner.

```python
# Define a hierarchy of custom exceptions
class InfrastructureError(Exception):
    """Base exception for all infrastructure-related errors."""
    pass

class ServerNotFoundError(InfrastructureError):
    """Raised when a server cannot be found."""
    
    def __init__(self, server_name, region=None):
        self.server_name = server_name
        self.region = region
        region_info = f" in region {region}" if region else ""
        super().__init__(f"Server '{server_name}' not found{region_info}")

class HealthCheckFailedError(InfrastructureError):
    """Raised when a health check fails after retries."""
    
    def __init__(self, server_name, attempts, last_error=None):
        self.server_name = server_name
        self.attempts = attempts
        self.last_error = last_error
        super().__init__(
            f"Health check failed for '{server_name}' after {attempts} attempts. "
            f"Last error: {last_error}"
        )

class ConfigValidationError(Exception):
    """Raised when a configuration file fails validation."""
    
    def __init__(self, field, message):
        self.field = field
        super().__init__(f"Configuration error in field '{field}': {message}")


# Usage
def find_server(name, region):
    servers = get_all_servers()  # hypothetical function
    for server in servers:
        if server.name == name and server.region == region:
            return server
    raise ServerNotFoundError(name, region)

def get_server_status(server_name, region):
    try:
        server = find_server(server_name, region)
        return check_health(server)
    except ServerNotFoundError as e:
        print(f"Cannot check status: {e}")
        print(f"Server name searched: {e.server_name}")
        return None
    except InfrastructureError as e:
        # Catch any other infra error
        print(f"Infrastructure error: {e}")
        return None
```

---

## Implementing Retry Logic

One of the most common patterns in DevOps scripting is retrying transient failures:

```python
import time
import random

def with_retry(func, max_attempts=3, base_delay=1.0, backoff_factor=2.0):
    """
    Retry a function with exponential backoff.
    
    Args:
        func: A callable (function) to retry.
        max_attempts: Maximum number of tries.
        base_delay: Initial wait time in seconds.
        backoff_factor: Multiply the delay by this after each failure.
    
    Returns:
        The result of func() if it succeeds.
    
    Raises:
        The last exception if all attempts fail.
    """
    last_exception = None
    
    for attempt in range(1, max_attempts + 1):
        try:
            return func()  # Try to call the function
        except (ConnectionError, TimeoutError) as e:
            last_exception = e
            if attempt == max_attempts:
                break   # Don't sleep after the last attempt
            
            # Exponential backoff with jitter (random small addition)
            delay = base_delay * (backoff_factor ** (attempt - 1))
            jitter = random.uniform(0, 0.1 * delay)  # 10% jitter
            wait_time = delay + jitter
            
            print(f"Attempt {attempt} failed: {e}. Retrying in {wait_time:.1f}s...")
            time.sleep(wait_time)
    
    raise last_exception  # All attempts failed — re-raise the last error


# Usage
def check_api():
    import requests
    response = requests.get("https://api.example.com/health", timeout=5)
    response.raise_for_status()
    return response.json()

try:
    result = with_retry(check_api, max_attempts=3, base_delay=2.0)
    print("API is healthy:", result)
except Exception as e:
    print(f"API health check failed after all retries: {e}")
```

---

## Context Managers: Beyond Files

The `with` statement works with any "context manager" — objects that define setup and teardown behaviour:

```python
import contextlib

@contextlib.contextmanager
def managed_connection(host, port):
    """
    A context manager that opens and closes a connection.
    Everything before 'yield' is setup; everything after is teardown.
    """
    print(f"Opening connection to {host}:{port}")
    connection = create_connection(host, port)  # hypothetical
    
    try:
        yield connection  # The 'with' block receives this value
    except Exception as e:
        print(f"Error during connection use: {e}")
        raise
    finally:
        print(f"Closing connection to {host}:{port}")
        connection.close()

# Usage
with managed_connection("db.prod.internal", 5432) as conn:
    results = conn.query("SELECT * FROM servers")
    print(results)
# Connection is automatically closed here
```

---

## Common Beginner Mistakes

**Mistake 1: Catching too broadly**

```python
# WRONG — hides ALL errors, including bugs in your code
try:
    do_something()
except Exception:
    pass  # Silently swallow the error

# CORRECT — only catch what you expect
try:
    data = json.load(f)
except json.JSONDecodeError as e:
    print(f"Invalid JSON: {e}")
    data = {}
```

**Mistake 2: Not using the exception object**

```python
# WRONG — loses the error message
except ValueError:
    print("A ValueError occurred")

# CORRECT — capture and use the message
except ValueError as e:
    print(f"ValueError: {e}")
    # e.args contains the message; str(e) gives the string representation
```

**Mistake 3: Catching and re-raising incorrectly**

```python
# WRONG — loses the original traceback (makes debugging hard)
try:
    risky_operation()
except Exception as e:
    raise Exception(f"Something went wrong: {e}")  # New exception, lost original

# CORRECT — preserve original context
try:
    risky_operation()
except Exception as e:
    raise RuntimeError("Something went wrong") from e  # Chains exceptions

# OR: just re-raise without creating a new exception
try:
    risky_operation()
except Exception:
    log_error()
    raise  # Re-raises the original exception with original traceback
```

---

## How This Works in the Real World

Professional DevOps scripts almost always have a structure like this:

```python
import sys
import logging

logging.basicConfig(level=logging.INFO, format="%(asctime)s %(levelname)s %(message)s")
log = logging.getLogger(__name__)

def main():
    try:
        config = load_config("config.yaml")
        validate_config(config)
        
        results = []
        errors = []
        
        for server in config["servers"]:
            try:
                result = check_server_health(server)
                results.append(result)
            except HealthCheckFailedError as e:
                log.error(f"Health check failed: {e}")
                errors.append({"server": server, "error": str(e)})
            except Exception as e:
                log.exception(f"Unexpected error checking {server}")  # logs full traceback
                errors.append({"server": server, "error": str(e)})
        
        generate_report(results, errors)
        
        # Exit with non-zero code if there were errors (important for CI/CD)
        if errors:
            sys.exit(1)
    
    except ConfigValidationError as e:
        log.error(f"Configuration is invalid: {e}")
        sys.exit(2)
    except KeyboardInterrupt:
        log.info("Interrupted by user")
        sys.exit(0)

if __name__ == "__main__":
    main()
```

---

## Chapter 4 Summary

- `try/except/else/finally` gives you full control over error handling
- Catch specific exception types — avoid bare `except:` or `except Exception:` without logging
- Use `as e` to capture the exception object and its message
- `else` runs only on success; `finally` always runs
- Create custom exception classes that carry contextual information
- Implement retry logic with exponential backoff for transient network errors
- Always `raise` from a caught exception when creating a new one, to preserve the chain
- Use `sys.exit(1)` to signal failure in CLI scripts



# Chapter 5: Working with the OS
## os, sys, pathlib, shutil, tempfile

---

## The Computer Beneath Your Script

Python doesn't run in a vacuum. It runs on an operating system, on a file system, with environment variables, processes, and users. A DevOps script often needs to interact with all of these: find files, check if paths exist, create directories, copy files, inspect the running process.

Python gives you several modules for this. We'll cover the most important ones in order of how often you'll use them.

---

## pathlib — The Modern Way to Handle Paths

`pathlib` was introduced in Python 3.4 and is now the recommended way to work with file system paths. It represents paths as objects rather than strings, which is both safer and more readable.

```python
from pathlib import Path

# Creating Path objects
home = Path.home()              # /home/ubuntu or /Users/yourname
cwd = Path.cwd()                # Current working directory
config_file = Path("/etc/app/config.yaml")
relative = Path("logs/app.log") # Relative path (from wherever script runs)

# Building paths with / operator (much cleaner than os.path.join)
log_dir = home / "logs"         # /home/ubuntu/logs
log_file = log_dir / "app.log"  # /home/ubuntu/logs/app.log

print(log_file)           # /home/ubuntu/logs/app.log (as string representation)
print(log_file.name)      # app.log (just the filename)
print(log_file.stem)      # app (filename without extension)
print(log_file.suffix)    # .log (the extension)
print(log_file.parent)    # /home/ubuntu/logs (the directory containing this file)
print(log_file.parts)     # ('/', 'home', 'ubuntu', 'logs', 'app.log')

# Checking if paths exist
print(config_file.exists())     # True or False
print(config_file.is_file())    # True if it's a file
print(config_file.is_dir())     # True if it's a directory

# Creating directories
output_dir = Path("/tmp/reports")
output_dir.mkdir(parents=True, exist_ok=True)
# parents=True: creates parent directories too (like mkdir -p)
# exist_ok=True: doesn't raise an error if the directory already exists

# Reading and writing files (Path objects work as file paths)
config_file = Path("/tmp/test-config.txt")
config_file.write_text("host=localhost\nport=5432\n")  # Write string to file
content = config_file.read_text()                       # Read entire file as string
print(content)

# Listing directory contents
log_dir = Path("/var/log")
for item in log_dir.iterdir():
    print(f"{'DIR' if item.is_dir() else 'FILE'}: {item.name}")

# Searching for files (glob patterns)
config_dir = Path("/etc")
for yaml_file in config_dir.glob("*.yaml"):   # All .yaml files in /etc
    print(yaml_file)

for log_file in Path("/var/log").rglob("*.log"):  # Recursive — all .log files
    print(log_file)

# Get absolute path
relative = Path("config.yaml")
absolute = relative.resolve()  # /home/ubuntu/myproject/config.yaml
```

---

## os — Interacting with the Operating System

```python
import os

# Environment variables
path = os.environ.get("PATH", "")          # Get with a default value
debug_mode = os.environ.get("DEBUG", "false").lower() == "true"
home_dir = os.environ["HOME"]              # Raises KeyError if not set

# Set environment variables (only affects current process)
os.environ["APP_ENV"] = "production"

# Current process information
print(os.getpid())    # Current process ID
print(os.getppid())   # Parent process ID
print(os.getcwd())    # Current working directory (as string)
os.chdir("/tmp")      # Change working directory

# File and directory operations
os.makedirs("/tmp/test/nested/dir", exist_ok=True)   # mkdir -p equivalent
os.rename("/tmp/old-name.txt", "/tmp/new-name.txt")  # Move/rename
os.remove("/tmp/file-to-delete.txt")                 # Delete a file
os.rmdir("/tmp/empty-dir")                           # Remove empty directory

# File statistics
stat = os.stat("/etc/hostname")
print(stat.st_size)                          # File size in bytes
print(stat.st_mtime)                         # Last modification time (unix timestamp)

import datetime
mtime = datetime.datetime.fromtimestamp(stat.st_mtime)
print(f"Last modified: {mtime}")

# Walking a directory tree (all files and subdirectories)
for dirpath, dirnames, filenames in os.walk("/etc"):
    print(f"In directory: {dirpath}")
    for filename in filenames:
        full_path = os.path.join(dirpath, filename)
        print(f"  File: {full_path}")
    # Modify dirnames IN PLACE to control which subdirs are visited
    dirnames[:] = [d for d in dirnames if not d.startswith(".")]  # Skip hidden dirs
```

---

## sys — The Running Python Process

```python
import sys

# Python version information
print(sys.version)          # "3.11.4 (main, ...)"
print(sys.version_info)     # sys.version_info(major=3, minor=11, micro=4, ...)
print(sys.version_info >= (3, 8))  # True — check minimum version

# Command-line arguments
# If you run: python script.py arg1 arg2 --flag
print(sys.argv)   # ['script.py', 'arg1', 'arg2', '--flag']
print(sys.argv[0])  # 'script.py' — always the script name
print(sys.argv[1:]) # ['arg1', 'arg2', '--flag'] — everything after script name

# Platform detection
print(sys.platform)  # 'linux', 'darwin' (macOS), 'win32' (Windows)

if sys.platform == "win32":
    config_dir = Path(os.environ.get("APPDATA", "")) / "myapp"
else:
    config_dir = Path.home() / ".config" / "myapp"

# Exit codes (important for scripts called by CI/CD or other scripts)
# 0 = success, non-zero = failure
# sys.exit() is better than just letting the script end — it's explicit

def main():
    success = run_checks()
    sys.exit(0 if success else 1)

# Standard streams (stdout, stderr)
print("This goes to stdout")
print("This is an error message", file=sys.stderr)  # Write to stderr

# Path manipulation
sys.path.insert(0, "/path/to/my/modules")  # Add directory to module search path
```

---

## shutil — High-Level File Operations

`shutil` (shell utilities) provides higher-level file operations — copying, moving, archiving:

```python
import shutil
from pathlib import Path

# Copying files
shutil.copy("/etc/nginx/nginx.conf", "/tmp/nginx.conf.bak")   # Copy file + permissions
shutil.copy2("/etc/nginx/nginx.conf", "/tmp/nginx.conf.bak2") # Copy file + metadata

# Copying entire directory trees
shutil.copytree("/etc/nginx", "/tmp/nginx-backup")
# exist_ok=True allows the destination to already exist (Python 3.8+)
shutil.copytree("/etc/nginx", "/tmp/nginx-backup-2", dirs_exist_ok=True)

# Moving files or directories
shutil.move("/tmp/old-dir", "/tmp/new-dir")

# Deleting entire directory trees (CAREFUL — this is not reversible!)
shutil.rmtree("/tmp/old-backup")
# Safe version: only delete if it exists
if Path("/tmp/old-backup").exists():
    shutil.rmtree("/tmp/old-backup")

# Creating archives
# Creates /tmp/nginx-backup.tar.gz from /etc/nginx/
shutil.make_archive(
    base_name="/tmp/nginx-backup",   # Output file name (without extension)
    format="gztar",                  # gz, bz2, xz, zip, tar
    root_dir="/etc",                 # Archive from this directory
    base_dir="nginx"                 # The directory within root_dir to archive
)

# Extracting archives
shutil.unpack_archive("/tmp/nginx-backup.tar.gz", "/tmp/nginx-restored/")

# Disk space information
total, used, free = shutil.disk_usage("/")
print(f"Total: {total // (1024**3)} GB")
print(f"Used:  {used  // (1024**3)} GB")
print(f"Free:  {free  // (1024**3)} GB")
print(f"Usage: {used/total*100:.1f}%")

# Finding executables (like `which` command)
python_path = shutil.which("python3")
print(f"python3 is at: {python_path}")  # /usr/bin/python3
```

---

## tempfile — Safe Temporary Files

When your script needs temporary files or directories, never hardcode `/tmp/myfile.txt`. Use `tempfile`, which creates unique names and handles cleanup:

```python
import tempfile
from pathlib import Path

# Create a temporary file (automatically deleted when closed in 'with' block)
with tempfile.NamedTemporaryFile(mode="w", suffix=".json", delete=True) as tmp:
    tmp.write('{"key": "value"}')
    tmp.flush()
    print(f"Temp file: {tmp.name}")  # Something like /tmp/tmpXXXXXX.json
    # File is deleted when the 'with' block exits

# Create a temporary file that persists after the 'with' block
# (You must delete it manually)
with tempfile.NamedTemporaryFile(mode="w", suffix=".txt", delete=False) as tmp:
    tmp.write("temporary content")
    temp_path = Path(tmp.name)  # Save the path before the 'with' block closes it

try:
    process_file(temp_path)  # Use the temp file
finally:
    temp_path.unlink(missing_ok=True)  # Clean up manually

# Create a temporary directory (everything inside is cleaned up automatically)
with tempfile.TemporaryDirectory() as tmpdir:
    tmpdir_path = Path(tmpdir)
    (tmpdir_path / "config.yaml").write_text("host: localhost")
    (tmpdir_path / "secrets.json").write_text('{"key": "secret"}')
    
    # Do work with files in tmpdir_path...
    print(f"Working in: {tmpdir}")
# Entire directory and contents are deleted here

# Get the system temp directory
print(tempfile.gettempdir())  # /tmp on Linux/Mac, %TEMP% on Windows
```

---

## How This Works in the Real World

Here's a real-world pattern: a script that backs up a configuration directory before making changes:

```python
import shutil
import tempfile
from pathlib import Path
from datetime import datetime

def backup_config(config_dir: str, backup_root: str = "/var/backups") -> Path:
    """
    Create a timestamped backup of a configuration directory.
    Returns the path to the backup archive.
    """
    config_path = Path(config_dir)
    backup_dir = Path(backup_root)
    
    if not config_path.exists():
        raise FileNotFoundError(f"Config directory not found: {config_dir}")
    
    backup_dir.mkdir(parents=True, exist_ok=True)
    
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    archive_name = f"{config_path.name}_{timestamp}"
    archive_path = backup_dir / archive_name
    
    shutil.make_archive(str(archive_path), "gztar", str(config_path.parent), config_path.name)
    
    final_path = Path(str(archive_path) + ".tar.gz")
    size_mb = final_path.stat().st_size / (1024 * 1024)
    
    print(f"Backed up {config_dir} to {final_path} ({size_mb:.1f} MB)")
    return final_path
```

---

## Chapter 5 Summary

- **`pathlib.Path`** is the modern, recommended way to work with file paths — use `/` to join, and methods like `.exists()`, `.mkdir()`, `.glob()`
- **`os`** provides OS-level interactions: environment variables, process info, low-level file operations, and `os.walk()` for directory traversal
- **`sys`** gives you access to the running Python process: command-line args (`sys.argv`), platform detection, exit codes (`sys.exit()`), and standard streams
- **`shutil`** handles high-level file operations: copying trees, moving, deleting trees, creating/extracting archives
- **`tempfile`** creates safe temporary files and directories with unique names — always use it instead of hardcoded `/tmp/` paths
- Prefer `pathlib` over `os.path` for all new code — it's cleaner and more readable

---

# Chapter 6: Subprocess and Shell Integration
## subprocess, Popen, shlex

---

## When Python Needs to Call the Shell

Python does most things well, but some tasks are best handled by existing shell tools: you might need to run `kubectl`, `terraform`, `git`, `curl`, `aws`, or any other command-line tool from within your Python script.

Think of Python as an orchestrator. It runs the right commands at the right time, captures their output, checks if they succeeded, and takes action based on the results.

This is what the `subprocess` module is for.

---

## The subprocess Module

### subprocess.run() — The Standard Way

`subprocess.run()` is the recommended function for running commands and waiting for them to finish:

```python
import subprocess

# Run a simple command
result = subprocess.run(["ls", "-la", "/tmp"])
# ls: the command
# -la: arguments (each as a separate list item)
# /tmp: the path argument

print(result.returncode)  # 0 means success, non-zero means failure

# Capture output (otherwise it goes straight to the terminal)
result = subprocess.run(
    ["hostname"],
    capture_output=True,   # Capture both stdout and stderr
    text=True              # Decode bytes to string (using system encoding)
)

print(result.stdout)       # "prod-server-01\n"
print(result.stderr)       # "" (empty if no errors)
print(result.returncode)   # 0

# Strip whitespace/newlines from output
hostname = result.stdout.strip()
print(f"Hostname: {hostname}")
```

### Checking for Errors

```python
import subprocess

# Method 1: Check returncode manually
result = subprocess.run(["ls", "/nonexistent-path"], capture_output=True, text=True)
if result.returncode != 0:
    print(f"Command failed: {result.stderr}")

# Method 2: check=True raises CalledProcessError automatically
try:
    result = subprocess.run(
        ["ls", "/nonexistent-path"],
        capture_output=True,
        text=True,
        check=True  # Raises subprocess.CalledProcessError if returncode != 0
    )
except subprocess.CalledProcessError as e:
    print(f"Command failed with exit code {e.returncode}")
    print(f"stderr: {e.stderr}")
    print(f"stdout: {e.stdout}")
```

### Setting Timeouts

```python
try:
    result = subprocess.run(
        ["curl", "https://slow-api.example.com"],
        capture_output=True,
        text=True,
        timeout=30  # Raise TimeoutExpired after 30 seconds
    )
except subprocess.TimeoutExpired as e:
    print(f"Command timed out after {e.timeout} seconds")
```

### Running Shell Commands with shell=True

Sometimes you want to run a shell command that uses pipes, redirections, or shell built-ins:

```python
# WARNING: shell=True is convenient but has security implications
# NEVER use it with untrusted user input

# OK for hardcoded commands
result = subprocess.run(
    "ps aux | grep python | grep -v grep",
    shell=True,           # Run through the shell (bash/sh)
    capture_output=True,
    text=True
)
print(result.stdout)

# DANGEROUS — never do this with user input
user_input = input("Enter filename: ")  # Could be "file.txt; rm -rf /"
subprocess.run(f"cat {user_input}", shell=True)  # SHELL INJECTION VULNERABILITY
```

### Environment Variables for Subprocesses

```python
import os
import subprocess

# Pass environment variables to a subprocess
env = os.environ.copy()      # Start with current environment
env["AWS_PROFILE"] = "prod"  # Add/override specific variables
env["AWS_REGION"] = "us-east-1"

result = subprocess.run(
    ["aws", "s3", "ls"],
    env=env,
    capture_output=True,
    text=True
)
```

---

## shlex — Safely Quoting Shell Commands

When building shell command strings from variables, `shlex` helps you quote arguments safely:

```python
import shlex

# shlex.split() parses a shell command string into a list (handles quotes correctly)
command = 'ls -la "/path with spaces/file name.txt"'
args = shlex.split(command)
print(args)  # ['ls', '-la', '/path with spaces/file name.txt']

# shlex.quote() escapes a string so it's safe to use as a shell argument
filename = "my file with spaces; rm -rf /"  # Dangerous input
safe = shlex.quote(filename)
print(safe)  # "'my file with spaces; rm -rf /'" — safely quoted

# Building commands safely
def run_git_command(repo_path, *git_args):
    """Safely run a git command in a specific directory."""
    cmd = ["git", "-C", str(repo_path)] + list(git_args)
    return subprocess.run(cmd, capture_output=True, text=True, check=True)

result = run_git_command("/home/ubuntu/myrepo", "status")
print(result.stdout)
```

---

## Popen — Non-Blocking and Streaming Output

`subprocess.run()` waits for the command to finish. `Popen` gives you more control: you can read output as it's produced (streaming), write to stdin, and run processes in parallel.

```python
import subprocess

# Stream output in real time (useful for long-running processes)
with subprocess.Popen(
    ["ping", "-c", "4", "8.8.8.8"],  # ping 4 times
    stdout=subprocess.PIPE,
    stderr=subprocess.PIPE,
    text=True
) as proc:
    for line in proc.stdout:        # Read one line at a time as it's produced
        print(f"[ping] {line.strip()}")
    
    proc.wait()  # Wait for process to finish
    if proc.returncode != 0:
        print(f"ping failed: {proc.stderr.read()}")

# Piping output between processes (like: ps aux | grep python)
ps_proc = subprocess.Popen(
    ["ps", "aux"],
    stdout=subprocess.PIPE
)
grep_proc = subprocess.Popen(
    ["grep", "python"],
    stdin=ps_proc.stdout,   # Connect ps's output to grep's input
    stdout=subprocess.PIPE,
    text=True
)
ps_proc.stdout.close()  # Allow ps_proc to receive SIGPIPE if grep_proc exits

output, _ = grep_proc.communicate()  # Wait for grep to finish and get output
print(output)
```

---

## A Practical Wrapper for Running Commands

Here's a reusable helper that most DevOps scripts benefit from:

```python
import subprocess
import logging
from typing import Optional, List, Dict

log = logging.getLogger(__name__)

def run_command(
    args: List[str],
    cwd: Optional[str] = None,
    env: Optional[Dict[str, str]] = None,
    timeout: int = 60,
    check: bool = True
) -> subprocess.CompletedProcess:
    """
    Run a shell command with consistent error handling and logging.
    
    Args:
        args: Command and arguments as a list.
        cwd: Working directory for the command.
        env: Environment variables (if None, inherits current environment).
        timeout: How long to wait before killing the process.
        check: If True, raise an exception if command fails.
    
    Returns:
        CompletedProcess object with .stdout, .stderr, .returncode
    """
    import os
    
    # If custom env provided, merge with current environment
    if env:
        full_env = os.environ.copy()
        full_env.update(env)
    else:
        full_env = None
    
    log.debug(f"Running: {' '.join(args)}")
    
    try:
        result = subprocess.run(
            args,
            capture_output=True,
            text=True,
            cwd=cwd,
            env=full_env,
            timeout=timeout,
            check=check
        )
        log.debug(f"Exit code: {result.returncode}")
        return result
    
    except subprocess.CalledProcessError as e:
        log.error(f"Command failed: {' '.join(args)}")
        log.error(f"Exit code: {e.returncode}")
        log.error(f"stderr: {e.stderr}")
        raise
    
    except subprocess.TimeoutExpired:
        log.error(f"Command timed out after {timeout}s: {' '.join(args)}")
        raise


# Usage
result = run_command(["terraform", "plan", "-out=plan.tfplan"], cwd="/home/ubuntu/infra")
print(result.stdout)
```

---

## Common Beginner Mistakes

**Mistake 1: Using shell=True with user-provided input**

```python
# NEVER do this — shell injection vulnerability
filename = request.get("filename")  # User-controlled input
subprocess.run(f"cat {filename}", shell=True)

# SAFE — always pass arguments as a list
subprocess.run(["cat", filename], capture_output=True)
```

**Mistake 2: Not capturing output**

```python
# This works, but output goes to the terminal and isn't captured
result = subprocess.run(["ls"])

# To use the output in your code, you must capture it
result = subprocess.run(["ls"], capture_output=True, text=True)
output = result.stdout
```

**Mistake 3: Forgetting text=True**

```python
result = subprocess.run(["hostname"], capture_output=True)
print(result.stdout)      # b'web-prod-01\n' — bytes object!
print(result.stdout.decode())  # "web-prod-01\n" — decoded manually

# Easier: use text=True upfront
result = subprocess.run(["hostname"], capture_output=True, text=True)
print(result.stdout.strip())   # "web-prod-01" — already a string
```

---

## Chapter 6 Summary

- Use `subprocess.run()` for most command execution — it waits for completion and returns results
- Pass commands as **lists** (not strings) whenever possible — safer and cleaner
- `capture_output=True` captures stdout and stderr; `text=True` decodes them to strings
- `check=True` auto-raises `CalledProcessError` on failure; otherwise check `.returncode`
- `timeout=N` prevents commands from hanging forever
- Use `shell=True` with caution — never with untrusted user input
- `shlex.split()` parses command strings; `shlex.quote()` safely quotes shell arguments
- Use `Popen` when you need streaming output or more control over I/O

---

# Chapter 7: HTTP Clients
## requests, httpx — GET, POST, Auth, Retries, Timeouts

---

## The World Runs on HTTP

Almost every infrastructure tool communicates via HTTP. AWS APIs, Kubernetes API server, Vault, Consul, Prometheus, Grafana, Slack, GitHub, PagerDuty — they all expose HTTP APIs. A DevOps engineer who can confidently make HTTP requests in Python can automate nearly anything.

---

## Installation

```bash
pip install requests httpx
```

---

## The requests Library

`requests` is the most widely used HTTP library in Python — clean, intuitive, and powerful.

### GET Requests

```python
import requests

# Basic GET request
response = requests.get("https://httpbin.org/get")

# Response attributes
print(response.status_code)    # 200
print(response.ok)             # True if status code is 2xx
print(response.headers)        # Response headers dict
print(response.text)           # Response body as string
print(response.json())         # Parse JSON response body (raises ValueError if not JSON)
print(response.content)        # Response body as bytes

# GET with query parameters
params = {
    "region": "us-east-1",
    "status": "running",
    "tag:env": "prod"
}
response = requests.get(
    "https://api.example.com/servers",
    params=params   # Appended as ?region=us-east-1&status=running&tag%3Aenv=prod
)
print(response.url)  # See the full URL with params
```

### POST Requests

```python
import requests

# POST with JSON body
payload = {
    "server_name": "web-new-01",
    "region": "us-east-1",
    "instance_type": "t3.medium"
}
response = requests.post(
    "https://api.example.com/servers",
    json=payload    # Automatically sets Content-Type: application/json and serialises payload
)
print(response.status_code)  # 201 Created
print(response.json())       # {"id": "srv-12345", "status": "provisioning"}

# POST with form data
response = requests.post(
    "https://api.example.com/login",
    data={"username": "admin", "password": "secret"}
    # data= sends as application/x-www-form-urlencoded
)

# POST with file upload
with open("config.yaml", "rb") as f:
    response = requests.post(
        "https://api.example.com/configs",
        files={"file": ("config.yaml", f, "application/x-yaml")}
    )
```

### Headers and Authentication

```python
import requests
import os

# Custom headers
headers = {
    "Authorization": f"Bearer {os.environ['API_TOKEN']}",
    "X-Request-ID": "req-12345",
    "Accept": "application/json"
}
response = requests.get("https://api.example.com/servers", headers=headers)

# HTTP Basic Auth
response = requests.get(
    "https://api.example.com/data",
    auth=("username", "password")  # Sends as HTTP Basic Auth header
)

# API Key in header (common pattern)
response = requests.get(
    "https://api.example.com/data",
    headers={"X-API-Key": os.environ["API_KEY"]}
)
```

### Timeouts — Always Set Them

Without a timeout, a request can hang forever. Always set a timeout:

```python
import requests

# Single value — applies to both connect and read timeout
response = requests.get("https://api.example.com", timeout=10)

# Tuple — (connect_timeout, read_timeout)
response = requests.get(
    "https://api.example.com",
    timeout=(5, 30)  # 5 seconds to connect, 30 seconds to read
)

# Handle timeout
try:
    response = requests.get("https://api.example.com", timeout=10)
    response.raise_for_status()  # Raises HTTPError for 4xx/5xx responses
except requests.exceptions.Timeout:
    print("Request timed out")
except requests.exceptions.HTTPError as e:
    print(f"HTTP error: {e.response.status_code} {e.response.reason}")
except requests.exceptions.ConnectionError:
    print("Failed to connect — network issue?")
```

### Sessions — Reusing Connections

A `Session` maintains connection pooling and shared settings (headers, auth) across multiple requests:

```python
import requests

# Without session: new connection for each request
for server in servers:
    response = requests.get(f"https://api.example.com/servers/{server}")

# With session: reuses connection, more efficient
with requests.Session() as session:
    session.headers.update({
        "Authorization": f"Bearer {token}",
        "Accept": "application/json"
    })
    
    for server in servers:
        response = session.get(f"https://api.example.com/servers/{server}")
        print(response.json())
```

### Retries with urllib3

```python
import requests
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

def create_session_with_retries(
    total_retries=3,
    backoff_factor=0.5,
    status_forcelist=(500, 502, 503, 504)
):
    """
    Create a requests Session that automatically retries on failure.
    
    Args:
        total_retries: Maximum number of retry attempts.
        backoff_factor: Sleep time multiplier between retries.
                        Retries at: backoff_factor * (2 ^ (retry_number - 1))
                        0.5 -> 0.5s, 1s, 2s, 4s...
        status_forcelist: HTTP status codes that should trigger a retry.
    """
    retry_strategy = Retry(
        total=total_retries,
        backoff_factor=backoff_factor,
        status_forcelist=status_forcelist,
        allowed_methods=["GET", "POST", "PUT", "DELETE"]
    )
    
    adapter = HTTPAdapter(max_retries=retry_strategy)
    
    session = requests.Session()
    session.mount("https://", adapter)
    session.mount("http://", adapter)
    
    return session

# Usage
session = create_session_with_retries()
try:
    response = session.get("https://api.example.com/status", timeout=10)
    response.raise_for_status()
    print(response.json())
except requests.exceptions.RetryError:
    print("All retries exhausted")
except requests.exceptions.HTTPError as e:
    print(f"Final HTTP error: {e}")
```

---

## httpx — The Modern Alternative

`httpx` is a newer library that is mostly compatible with `requests` but adds:
- **Async support** (crucial for concurrent HTTP requests — see Chapter 15)
- **HTTP/2** support
- **Built-in timeout management**

```python
import httpx

# Synchronous usage (almost identical to requests)
with httpx.Client(timeout=10.0) as client:
    response = client.get("https://httpbin.org/get")
    print(response.json())

# The key difference: async support for concurrent requests
import asyncio
import httpx

async def check_multiple_servers(urls):
    """Check multiple servers concurrently using async httpx."""
    async with httpx.AsyncClient(timeout=10.0) as client:
        tasks = [client.get(url) for url in urls]
        responses = await asyncio.gather(*tasks, return_exceptions=True)
        
        results = []
        for url, response in zip(urls, responses):
            if isinstance(response, Exception):
                results.append({"url": url, "healthy": False, "error": str(response)})
            else:
                results.append({
                    "url": url,
                    "healthy": response.status_code == 200,
                    "status": response.status_code
                })
        return results

# Run the async function
urls = [
    "https://api.example.com/health",
    "https://api2.example.com/health",
    "https://api3.example.com/health",
]
results = asyncio.run(check_multiple_servers(urls))
for r in results:
    status = "OK" if r["healthy"] else "FAIL"
    print(f"[{status}] {r['url']}")
```

---

## Handling API Responses Properly

```python
import requests
import json

def call_api(url: str, method: str = "GET", **kwargs) -> dict:
    """
    Make an API call with consistent error handling.
    Returns parsed JSON response or raises an appropriate exception.
    """
    try:
        response = requests.request(method, url, timeout=30, **kwargs)
        
        # Raise an exception for 4xx and 5xx status codes
        response.raise_for_status()
        
        # Parse JSON response
        content_type = response.headers.get("Content-Type", "")
        if "application/json" in content_type:
            return response.json()
        else:
            return {"raw": response.text, "status": response.status_code}
    
    except requests.exceptions.ConnectionError:
        raise RuntimeError(f"Cannot connect to {url}")
    
    except requests.exceptions.Timeout:
        raise TimeoutError(f"Request to {url} timed out")
    
    except requests.exceptions.HTTPError as e:
        status = e.response.status_code
        try:
            body = e.response.json()
        except (ValueError, json.JSONDecodeError):
            body = e.response.text
        
        if status == 401:
            raise PermissionError(f"Authentication failed for {url}")
        elif status == 403:
            raise PermissionError(f"Authorisation denied for {url}")
        elif status == 404:
            raise FileNotFoundError(f"Resource not found: {url}")
        elif status == 429:
            raise RuntimeError(f"Rate limit exceeded for {url}")
        else:
            raise RuntimeError(f"API error {status}: {body}")
```

---

## Chapter 7 Summary

- `requests` is the standard library for synchronous HTTP; `httpx` adds async support
- Always set `timeout` — without it, requests can hang forever
- Use `response.raise_for_status()` to automatically raise on 4xx/5xx responses
- Use `Session` for multiple requests to the same server — connection reuse is more efficient
- Add retry logic with `Retry` and `HTTPAdapter` for transient failures
- `json=payload` in requests sets Content-Type and serialises automatically
- Never hardcode credentials — use environment variables or config files

---

# Chapter 8: Environment and Configuration
## os.environ, python-dotenv, configparser

---

## The Problem with Hardcoded Configuration

Imagine you write a script to query your AWS account:

```python
# WRONG — NEVER do this
aws_access_key = "AKIAIOSFODNN7EXAMPLE"
aws_secret_key = "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
database_password = "my-super-secret-password"
```

Now your code has a secret baked in. If you commit this to git, that secret is in version history forever, even if you delete it later. This has caused massive security breaches at major companies.

The solution is **externalising configuration** — keeping secrets and environment-specific settings outside your code.

---

## Environment Variables — The Universal Config Mechanism

Environment variables are key-value pairs that exist in the shell environment. They're the most universal configuration mechanism: supported by every OS, CI/CD platform, container runtime, and cloud service.

```python
import os

# Reading environment variables
api_key = os.environ["API_KEY"]           # Raises KeyError if not set
api_key = os.environ.get("API_KEY")       # Returns None if not set
api_key = os.environ.get("API_KEY", "")   # Returns "" if not set

# Safe pattern: fail fast with a helpful error if required vars are missing
def require_env(name: str) -> str:
    """Get an environment variable or raise a clear error if missing."""
    value = os.environ.get(name)
    if value is None:
        raise EnvironmentError(
            f"Required environment variable '{name}' is not set. "
            f"Set it with: export {name}=your_value"
        )
    return value

api_key = require_env("API_KEY")
db_url = require_env("DATABASE_URL")

# Type conversion (environment variables are always strings)
debug = os.environ.get("DEBUG", "false").lower() in ("true", "1", "yes")
port = int(os.environ.get("PORT", "8080"))
workers = int(os.environ.get("WORKERS", "4"))
timeout = float(os.environ.get("TIMEOUT", "30.0"))

# Setting environment variables (only affects current process)
os.environ["COMPUTED_VALUE"] = "something"
```

---

## .env Files and python-dotenv

Running `export API_KEY=xxx` every time you open a terminal is tedious. `.env` files let you store development environment variables in a file:

```bash
# .env file — NEVER commit this to git!
API_KEY=my-development-api-key
DATABASE_URL=postgresql://localhost/devdb
DEBUG=true
LOG_LEVEL=DEBUG
AWS_PROFILE=dev
SLACK_WEBHOOK_URL=https://hooks.slack.com/services/xxx/yyy/zzz
```

```python
# Install: pip install python-dotenv
from dotenv import load_dotenv
import os

# Load variables from .env into os.environ
load_dotenv()  # Looks for .env in current directory and parent dirs

# Now they're available as normal environment variables
api_key = os.environ.get("API_KEY")
debug = os.environ.get("DEBUG", "false").lower() == "true"

# Specify a custom path
load_dotenv("/etc/app/.env.production")

# Override existing environment variables (useful in testing)
load_dotenv(override=True)

# dotenv_values() returns a dict without modifying os.environ
from dotenv import dotenv_values
config = dotenv_values(".env")
print(config["API_KEY"])
```

**Critical: Add `.env` to `.gitignore`:**

```bash
# .gitignore
.env
.env.local
.env.*.local
*.secret
```

---

## configparser — INI-Style Configuration Files

`configparser` handles `.ini` style configuration files — common for older tools, some monitoring systems, and Python applications:

```ini
# config.ini
[DEFAULT]
log_level = INFO
timeout = 30

[database]
host = localhost
port = 5432
name = appdb
user = appuser

[monitoring]
enabled = true
interval = 60
endpoints = https://api1.example.com, https://api2.example.com
```

```python
import configparser

config = configparser.ConfigParser()
config.read("config.ini")

# Reading values (always returns strings)
db_host = config["database"]["host"]      # "localhost"
db_port = config["database"]["port"]      # "5432" — string!
db_port = config.getint("database", "port")   # 5432 — integer
timeout = config.getfloat("DEFAULT", "timeout")  # 30.0 — float
enabled = config.getboolean("monitoring", "enabled")  # True — bool

# Checking if sections/keys exist
if config.has_section("database"):
    if config.has_option("database", "password"):
        db_pass = config["database"]["password"]

# DEFAULT section: values here are available in ALL sections
log_level = config["database"]["log_level"]  # "INFO" (from DEFAULT)

# Iterating over sections
for section in config.sections():
    print(f"\n[{section}]")
    for key, value in config[section].items():
        print(f"  {key} = {value}")

# Writing a config file
new_config = configparser.ConfigParser()
new_config["app"] = {
    "name": "health-checker",
    "version": "1.0.0"
}
new_config["logging"] = {
    "level": "INFO",
    "file": "/var/log/health-checker.log"
}

with open("app.ini", "w") as f:
    new_config.write(f)
```

---

## Building a Robust Config Loader

In real projects, you'll want a config system that:
1. Has sensible defaults
2. Reads from a config file
3. Allows environment variable overrides (for containers and CI/CD)
4. Validates required fields
5. Is type-safe

```python
from dataclasses import dataclass, field
from pathlib import Path
from typing import List, Optional
import os
import yaml
from dotenv import load_dotenv

@dataclass
class DatabaseConfig:
    host: str = "localhost"
    port: int = 5432
    name: str = "appdb"
    user: str = "appuser"
    password: str = ""

@dataclass
class AppConfig:
    environment: str = "development"
    debug: bool = False
    log_level: str = "INFO"
    database: DatabaseConfig = field(default_factory=DatabaseConfig)
    allowed_regions: List[str] = field(default_factory=lambda: ["us-east-1"])
    api_key: Optional[str] = None
    
    def __post_init__(self):
        """Validate the config after loading."""
        valid_envs = {"development", "staging", "production"}
        if self.environment not in valid_envs:
            raise ValueError(f"environment must be one of {valid_envs}")
        
        valid_log_levels = {"DEBUG", "INFO", "WARNING", "ERROR", "CRITICAL"}
        if self.log_level.upper() not in valid_log_levels:
            raise ValueError(f"log_level must be one of {valid_log_levels}")
        
        if self.environment == "production" and not self.api_key:
            raise ValueError("api_key is required in production")


def load_app_config(config_path: str = "config.yaml") -> AppConfig:
    """
    Load application configuration with the following priority (highest wins):
    1. Environment variables
    2. Config file
    3. Defaults
    """
    # Load .env file first (won't override existing env vars)
    load_dotenv()
    
    # Start with defaults
    raw_config = {}
    
    # Load from config file if it exists
    path = Path(config_path)
    if path.exists():
        with open(path) as f:
            raw_config = yaml.safe_load(f) or {}
    
    # Apply environment variable overrides
    db_config_data = raw_config.get("database", {})
    db_config_data.update({
        k: v for k, v in {
            "host":     os.environ.get("DB_HOST"),
            "port":     int(os.environ["DB_PORT"]) if "DB_PORT" in os.environ else None,
            "name":     os.environ.get("DB_NAME"),
            "user":     os.environ.get("DB_USER"),
            "password": os.environ.get("DB_PASSWORD"),
        }.items() if v is not None
    })
    
    return AppConfig(
        environment=os.environ.get("APP_ENV", raw_config.get("environment", "development")),
        debug=os.environ.get("DEBUG", str(raw_config.get("debug", False))).lower() in ("true", "1"),
        log_level=os.environ.get("LOG_LEVEL", raw_config.get("log_level", "INFO")).upper(),
        database=DatabaseConfig(**db_config_data),
        api_key=os.environ.get("API_KEY", raw_config.get("api_key")),
    )


# Usage
try:
    config = load_app_config()
    print(f"Environment: {config.environment}")
    print(f"Database: {config.database.host}:{config.database.port}")
except ValueError as e:
    print(f"Configuration error: {e}")
    import sys
    sys.exit(1)
```

---

## Chapter 8 Summary

- **Never** hardcode secrets or environment-specific values in code — always externalise them
- `os.environ` is the fundamental interface to environment variables
- Use `os.environ.get("KEY", "default")` to avoid `KeyError`; fail fast for required vars
- **python-dotenv** loads `.env` files into environment variables — add `.env` to `.gitignore`
- **configparser** handles `.ini` files — use `getint()`, `getboolean()`, `getfloat()` for type conversion
- Build a layered config system: defaults → file → environment variables (env wins)
- Validate configuration at startup — fail with a clear error rather than silently misconfigurate

---

# Chapter 9: Virtual Environments
## venv, pip, requirements.txt, pip-tools

---

## The Dependency Problem

Imagine this situation: You have two Python projects. Project A needs `requests==2.28.0`, but Project B needs `requests==2.31.0`. If you install them both globally, one will always win and the other will break.

This is the **dependency conflict problem**, and it's the reason virtual environments exist.

A **virtual environment** is an isolated Python installation for a single project. It has its own copy of Python and its own set of installed packages, completely separate from your system Python and from other projects.

---

## venv — Creating and Using Virtual Environments

`venv` is built into Python 3 — no installation needed:

```bash
# Create a virtual environment named 'venv' in the current directory
python3 -m venv venv
# 'm venv' tells Python to run the venv module
# 'venv' at the end is the name of the directory to create

# What this creates:
# venv/
#   bin/          (or Scripts/ on Windows)
#     python3     (symlink to Python)
#     pip3        (pip for this environment)
#     activate    (activation script)
#   lib/
#     python3.x/
#       site-packages/  (packages install here)

# Activate the virtual environment
source venv/bin/activate         # Linux/macOS
# venv\Scripts\activate.bat      # Windows CMD
# venv\Scripts\Activate.ps1      # Windows PowerShell

# Your prompt changes to show the active environment:
# (venv) ubuntu@server:~/myproject$

# Now pip installs into THIS environment, not globally
pip install requests

# Deactivate when done
deactivate
```

### Python in Scripts

When writing scripts that will be run directly, tell them to use the virtual environment's Python:

```bash
#!/usr/bin/env python3
# The above line makes this script use whichever python3 is active in the environment
```

---

## pip — Package Installation

```bash
# Install a package
pip install requests

# Install a specific version
pip install requests==2.31.0

# Install with minimum version constraint
pip install "requests>=2.28.0"

# Install multiple packages
pip install requests boto3 click pyyaml

# Upgrade a package to latest
pip install --upgrade requests

# Uninstall
pip uninstall requests

# See what's installed
pip list
pip show requests      # Details about a specific package

# Search for packages (requires internet)
pip index versions requests   # All available versions of requests
```

---

## requirements.txt — Recording Dependencies

A `requirements.txt` file records exactly which packages (and versions) your project needs:

```bash
# Generate requirements.txt from what's currently installed in your environment
pip freeze > requirements.txt

# This creates something like:
# boto3==1.34.0
# botocore==1.34.0
# certifi==2023.11.17
# requests==2.31.0
# urllib3==2.1.0
# ... (all transitive dependencies too)

# Install from requirements.txt (on another machine or in CI/CD)
pip install -r requirements.txt
```

### The Problem with pip freeze

`pip freeze` records **every** installed package, including transitive dependencies (dependencies of your dependencies). This creates a `requirements.txt` with 50+ packages when you only directly depend on 5. It also hardcodes exact versions, which can cause issues when packages release security patches.

A better approach distinguishes between **direct** and **transitive** dependencies:

```
# requirements.in — only YOUR direct dependencies
requests>=2.28.0
boto3>=1.30.0
click>=8.1.0
pyyaml>=6.0.0
python-dotenv>=1.0.0
```

---

## pip-tools — The Professional Approach

`pip-tools` solves the `requirements.txt` management problem properly:

```bash
# Install pip-tools
pip install pip-tools

# Create requirements.in with your DIRECT dependencies only
# (See the format above)

# Compile requirements.in into a fully pinned requirements.txt
pip-compile requirements.in
# Creates requirements.txt with ALL packages (direct + transitive) pinned to exact versions
# AND includes comments showing which packages require which other packages

# Install from the compiled requirements.txt
pip-sync requirements.txt  
# pip-sync also REMOVES packages not in requirements.txt (unlike pip install -r)

# Update all packages to latest compatible versions
pip-compile --upgrade requirements.in

# Update a specific package
pip-compile --upgrade-package requests requirements.in

# For separate dev dependencies
# Create requirements-dev.in:
# -r requirements.in   (includes all production deps)
# pytest>=7.0
# black>=23.0
# ruff>=0.1.0

pip-compile requirements-dev.in -o requirements-dev.txt
```

---

## Project Structure Best Practices

```
myproject/
├── venv/                    # Virtual environment (in .gitignore)
├── src/
│   └── myproject/
│       ├── __init__.py
│       └── main.py
├── tests/
│   └── test_main.py
├── requirements.in          # Direct production dependencies
├── requirements.txt         # Compiled, pinned (commit this)
├── requirements-dev.in      # Direct dev dependencies
├── requirements-dev.txt     # Compiled dev deps (commit this)
├── pyproject.toml           # Project metadata and tool config
├── .env                     # Environment variables (in .gitignore)
├── .env.example             # Template with dummy values (commit this)
└── .gitignore               # Must include venv/ and .env
```

**.gitignore essentials:**

```gitignore
# Virtual environments
venv/
.venv/
env/
.env/

# Secrets
.env
.env.local

# Python cache
__pycache__/
*.py[cod]
*.egg-info/
.pytest_cache/
.mypy_cache/
dist/
build/
```

---

## Checking What's Installed and Why

```bash
# Show a dependency tree — why is each package installed?
pip install pipdeptree
pipdeptree

# Example output:
# requests==2.31.0
#   - certifi [required: >=2017.4.17, installed: 2023.11.17]
#   - charset-normalizer [required: >=2,<4, installed: 3.3.2]
#   - idna [required: >=2.5,<4, installed: 3.6]
#   - urllib3 [required: >=1.21.1,<3, installed: 2.1.0]
```

---

## Common Beginner Mistakes

**Mistake 1: Installing packages globally**

```bash
# WRONG — installs into system Python
pip install requests

# CORRECT — always activate your venv first
source venv/bin/activate
pip install requests
```

**Mistake 2: Committing the venv directory**

The `venv/` directory is large, platform-specific, and machine-specific. Never commit it. Instead, commit `requirements.txt` and recreate the venv on each machine.

**Mistake 3: Not pinning versions in production**

```
# Unpinned — what version installs today might be different in 6 months
requests

# Pinned — reproducible builds
requests==2.31.0
```

For production scripts and tools, always pin your dependency versions in `requirements.txt`.

---

## How This Works in the Real World

In a CI/CD pipeline, you typically see something like:

```yaml
# .github/workflows/ci.yml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: "3.11"
      
      - name: Create virtual environment and install dependencies
        run: |
          python -m venv venv
          source venv/bin/activate
          pip install --upgrade pip
          pip install -r requirements.txt
          pip install -r requirements-dev.txt
      
      - name: Run tests
        run: |
          source venv/bin/activate
          pytest tests/
```

The key insight: the CI pipeline creates a fresh venv, installs exact pinned dependencies, and runs tests — producing the same result every time, on every machine.

---

## Chapter 9 Summary

- A **virtual environment** isolates a project's Python packages from the system and other projects
- Create with `python3 -m venv venv`; activate with `source venv/bin/activate`
- `pip install` installs packages; `pip freeze > requirements.txt` records them
- **pip-tools**: `requirements.in` for direct deps → `pip-compile` → `requirements.txt` for pinned deps
- **Always** add `venv/` and `.env` to `.gitignore`
- In CI/CD and production, always use pinned versions for reproducible builds
- `pip-sync` (from pip-tools) installs exactly what's in requirements.txt and removes anything extra



# Chapter 10: Package Management
## setuptools, pyproject.toml, Publishing to PyPI

---

## From Script to Shareable Package

So far, the code we've written lives in files you run directly. But what if you build a useful tool — a DNS checker, a cost report generator, a server health checker — that your whole team (or the world) should be able to use?

Python's package ecosystem lets you bundle your code, give it a name, specify its dependencies, and publish it so anyone can install it with `pip install your-tool-name`.

---

## pyproject.toml — The Modern Standard

`pyproject.toml` is the single configuration file for modern Python projects. It replaced the older `setup.py` and `setup.cfg` approach. Here's a complete example:

```toml
# pyproject.toml
[build-system]
requires = ["setuptools>=68.0", "wheel"]
build-backend = "setuptools.backends.legacy:build"
# 'build-system' tells pip how to build your package
# 'requires' are tools needed to BUILD (not runtime deps)
# 'build-backend' is the tool that does the actual building

[project]
name = "devops-toolkit"           # The name users will pip install
version = "1.2.0"                 # Semantic versioning: major.minor.patch
description = "A collection of DevOps automation tools"
readme = "README.md"              # Points to your README file
license = {text = "MIT"}
requires-python = ">=3.9"         # Minimum Python version required

# Your actual runtime dependencies
dependencies = [
    "requests>=2.28.0",
    "boto3>=1.30.0",
    "click>=8.1.0",
    "pyyaml>=6.0.0",
    "python-dotenv>=1.0.0",
]

# Project authors
[[project.authors]]
name = "Your Name"
email = "you@example.com"

# Classifiers help users find your package on PyPI
classifiers = [
    "Development Status :: 4 - Beta",
    "Environment :: Console",
    "Intended Audience :: Developers",
    "License :: OSI Approved :: MIT License",
    "Operating System :: OS Independent",
    "Programming Language :: Python :: 3",
    "Programming Language :: Python :: 3.9",
    "Programming Language :: Python :: 3.10",
    "Programming Language :: Python :: 3.11",
    "Topic :: System :: Systems Administration",
]

# Keywords help with PyPI search
keywords = ["devops", "automation", "cloud", "aws", "cli"]

# URLs shown on the PyPI page
[project.urls]
Homepage = "https://github.com/yourusername/devops-toolkit"
Documentation = "https://devops-toolkit.readthedocs.io"
"Bug Tracker" = "https://github.com/yourusername/devops-toolkit/issues"
Changelog = "https://github.com/yourusername/devops-toolkit/CHANGELOG.md"

# Optional dependencies (pip install devops-toolkit[dev])
[project.optional-dependencies]
dev = [
    "pytest>=7.0",
    "pytest-cov>=4.0",
    "black>=23.0",
    "ruff>=0.1.0",
    "mypy>=1.0",
    "pip-tools>=7.0",
]
docs = [
    "mkdocs>=1.5",
    "mkdocs-material>=9.0",
]

# Entry points: command-line scripts installed with the package
[project.scripts]
devops-toolkit = "devops_toolkit.cli:main"
# Users can run: devops-toolkit --help
# This runs the 'main' function in devops_toolkit/cli.py

# Tool-specific configuration
[tool.black]
line-length = 88
target-version = ["py39", "py310", "py311"]

[tool.ruff]
line-length = 88
target-version = "py39"
select = ["E", "F", "I", "N", "W", "UP"]   # Rule sets to check
ignore = ["E501"]                            # Rules to ignore

[tool.mypy]
python_version = "3.9"
strict = true
ignore_missing_imports = true

[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = "-v --cov=src/devops_toolkit --cov-report=term-missing"
```

---

## Project Structure for a Package

```
devops-toolkit/
├── src/
│   └── devops_toolkit/          # Your package code
│       ├── __init__.py          # Makes this a Python package
│       ├── cli.py               # CLI entry point
│       ├── health_check.py
│       └── aws/
│           ├── __init__.py
│           └── ec2.py
├── tests/
│   ├── __init__.py
│   ├── test_health_check.py
│   └── test_cli.py
├── docs/
│   └── index.md
├── pyproject.toml
├── README.md
├── CHANGELOG.md
├── LICENSE
├── .gitignore
└── requirements-dev.txt
```

The `src/` layout (placing your package inside a `src/` directory) is the recommended modern approach. It prevents accidentally importing from your source tree instead of the installed package during testing.

---

## Building and Publishing

### Building the Package

```bash
# Install build tools
pip install build twine

# Build the package — creates dist/ directory
python -m build
# Creates:
# dist/
#   devops_toolkit-1.2.0-py3-none-any.whl   (wheel — fast install)
#   devops_toolkit-1.2.0.tar.gz              (source distribution)
```

A **wheel** (`.whl`) is a pre-built package — fast to install. A **source distribution** (`.tar.gz`) includes your raw source code.

### Publishing to TestPyPI First

Always test your upload on TestPyPI (a separate test instance of PyPI) before publishing to the real PyPI:

```bash
# Upload to TestPyPI
twine upload --repository testpypi dist/*
# You'll be prompted for your TestPyPI credentials

# Install from TestPyPI to verify it works
pip install --index-url https://test.pypi.org/simple/ --extra-index-url https://pypi.org/simple/ devops-toolkit
# --extra-index-url pypi.org/simple: allows finding regular dependencies on real PyPI

# Once tested, publish to real PyPI
twine upload dist/*
```

### Using API Tokens (Recommended over username/password)

Create a `.pypirc` file in your home directory:

```ini
[distutils]
index-servers =
    pypi
    testpypi

[pypi]
repository = https://upload.pypi.org/legacy/
username = __token__
password = pypi-AgEIcHlwaS...  (your API token)

[testpypi]
repository = https://test.pypi.org/legacy/
username = __token__
password = pypi-AgEIcHlwaS...  (your TestPyPI token)
```

---

## Versioning: Semantic Versioning

Use **Semantic Versioning** (SemVer): `MAJOR.MINOR.PATCH`

- **PATCH** (`1.2.0` → `1.2.1`): Bug fixes only — nothing new, nothing broken
- **MINOR** (`1.2.0` → `1.3.0`): New features added in a backward-compatible way
- **MAJOR** (`1.2.0` → `2.0.0`): Breaking changes — existing code may need updates

```python
# src/devops_toolkit/__init__.py
__version__ = "1.2.0"
__author__ = "Your Name"
__description__ = "DevOps automation toolkit"
```

---

## Chapter 10 Summary

- `pyproject.toml` is the single configuration file for modern Python packages
- The `[project]` section defines metadata; `[build-system]` defines how to build
- `[project.scripts]` registers CLI commands that users can run after `pip install`
- Use `python -m build` to create wheel and source distribution
- Always test on **TestPyPI** before publishing to the real PyPI
- Use **Semantic Versioning**: MAJOR.MINOR.PATCH
- The `src/` layout prevents import confusion during development

---

# Chapter 11: CLI Development
## argparse, click, typer

---

## Turning Scripts into Tools

A script you run once or twice is fine as a file you call directly. But when a script grows into something your team uses regularly — with multiple modes, options, and subcommands — it deserves a proper command-line interface.

A good CLI looks like this:

```bash
$ health-checker --env prod --region us-east-1 --format json
$ health-checker servers list --status running
$ health-checker report --output /tmp/report.json --send-slack
```

---

## argparse — Standard Library CLI

`argparse` is built into Python and handles the basics well:

```python
import argparse
import sys

def build_parser():
    parser = argparse.ArgumentParser(
        description="Server health checker for DevOps teams",
        formatter_class=argparse.RawDescriptionHelpFormatter,
        epilog="""
Examples:
  %(prog)s --env prod
  %(prog)s --env staging --region us-west-2 --format json
  %(prog)s --env prod --verbose --output report.json
        """
    )
    
    parser.add_argument(
        "--env",
        choices=["dev", "staging", "prod"],
        required=True,
        help="Environment to check"
    )
    
    parser.add_argument(
        "--region",
        default="us-east-1",
        help="AWS region to check (default: %(default)s)"
    )
    
    parser.add_argument(
        "--format",
        choices=["json", "table", "csv"],
        default="table",
        help="Output format (default: %(default)s)"
    )
    
    parser.add_argument(
        "--output", "-o",
        type=str,
        metavar="FILE",
        help="Write output to FILE instead of stdout"
    )
    
    parser.add_argument(
        "--verbose", "-v",
        action="store_true",   # Flag: present = True, absent = False
        help="Enable verbose output"
    )
    
    parser.add_argument(
        "--timeout",
        type=int,
        default=10,
        help="Request timeout in seconds (default: %(default)s)"
    )
    
    return parser

def main():
    parser = build_parser()
    args = parser.parse_args()
    
    # args is a Namespace object — access parsed values as attributes
    print(f"Checking {args.env} environment in {args.region}")
    print(f"Format: {args.format}, Verbose: {args.verbose}")

if __name__ == "__main__":
    main()
```

---

## click — The Better CLI Library

`click` (install: `pip install click`) is significantly more ergonomic than `argparse`. It uses decorators to define your CLI:

```python
import click
import json
import sys

@click.group()
@click.option("--verbose/--no-verbose", default=False, help="Enable verbose output")
@click.pass_context
def cli(ctx, verbose):
    """
    DevOps Toolkit — Infrastructure automation for cloud engineers.
    
    Run a command with --help to see its options.
    """
    # ctx.ensure_object(dict) creates a shared context dict
    ctx.ensure_object(dict)
    ctx.obj["verbose"] = verbose


@cli.command()
@click.option("--env", "-e",
              type=click.Choice(["dev", "staging", "prod"], case_sensitive=False),
              required=True,
              help="Environment to check")
@click.option("--region", "-r",
              default="us-east-1",
              show_default=True,
              help="AWS region")
@click.option("--format", "-f",
              "output_format",   # Use 'output_format' as Python variable name
              type=click.Choice(["json", "table", "csv"]),
              default="table",
              show_default=True,
              help="Output format")
@click.option("--output", "-o",
              type=click.Path(writable=True),
              help="Write output to file")
@click.option("--timeout",
              type=click.IntRange(1, 300),
              default=10,
              show_default=True,
              help="Request timeout in seconds")
@click.pass_context
def check(ctx, env, region, output_format, output, timeout):
    """Check health of all servers in an environment."""
    verbose = ctx.obj.get("verbose", False)
    
    if verbose:
        click.echo(f"Checking {env} environment in {region}...")
    
    # Simulate health check
    results = [
        {"server": "web-01", "status": "healthy", "latency_ms": 45},
        {"server": "web-02", "status": "healthy", "latency_ms": 52},
        {"server": "db-01",  "status": "unhealthy", "latency_ms": None},
    ]
    
    # Format output
    if output_format == "json":
        output_text = json.dumps(results, indent=2)
    elif output_format == "csv":
        output_text = "server,status,latency_ms\n"
        for r in results:
            output_text += f"{r['server']},{r['status']},{r.get('latency_ms', '')}\n"
    else:  # table
        output_text = f"{'Server':<20} {'Status':<12} {'Latency':<10}\n"
        output_text += "-" * 44 + "\n"
        for r in results:
            latency = f"{r['latency_ms']}ms" if r['latency_ms'] else "N/A"
            color = "green" if r["status"] == "healthy" else "red"
            output_text += f"{r['server']:<20} {click.style(r['status'], fg=color):<12} {latency:<10}\n"
    
    # Write output
    if output:
        with open(output, "w") as f:
            f.write(output_text)
        click.echo(f"Results written to {output}")
    else:
        click.echo(output_text)
    
    # Exit with non-zero if any server is unhealthy
    unhealthy = [r for r in results if r["status"] != "healthy"]
    if unhealthy:
        click.echo(f"\n{len(unhealthy)} unhealthy server(s) found", err=True)
        sys.exit(1)


@cli.command()
@click.argument("server_name")
@click.option("--force", is_flag=True, help="Skip confirmation prompt")
def restart(server_name, force):
    """Restart a server by name."""
    if not force:
        # Prompt for confirmation
        click.confirm(f"Restart server '{server_name}'?", abort=True)
        # abort=True raises click.Abort (exits cleanly) if user says no
    
    click.echo(f"Restarting {server_name}...")


if __name__ == "__main__":
    cli()
```

**Key click features used here:**
- `@click.group()` creates a command group (supports subcommands like `git commit`, `git push`)
- `@click.pass_context` and `ctx.obj` share state between commands
- `click.Choice()` restricts to valid values
- `click.Path()` validates file paths
- `click.IntRange()` validates numeric ranges
- `click.style()` adds terminal colours
- `click.confirm()` prompts for confirmation

---

## typer — CLI from Type Hints

`typer` (install: `pip install typer`) generates CLI interfaces from Python type annotations. It's the most modern approach and produces clean, documented CLIs with minimal code:

```python
import typer
from enum import Enum
from typing import Optional
from pathlib import Path

app = typer.Typer(help="DevOps Toolkit — Infrastructure automation")

class OutputFormat(str, Enum):
    json = "json"
    table = "table"
    csv = "csv"

class Environment(str, Enum):
    dev = "dev"
    staging = "staging"
    prod = "prod"


@app.command()
def check(
    env: Environment = typer.Option(..., "--env", "-e", help="Environment to check"),
    region: str = typer.Option("us-east-1", "--region", "-r", help="AWS region"),
    output_format: OutputFormat = typer.Option(OutputFormat.table, "--format", "-f"),
    output: Optional[Path] = typer.Option(None, "--output", "-o", help="Output file"),
    timeout: int = typer.Option(10, "--timeout", min=1, max=300, help="Timeout in seconds"),
    verbose: bool = typer.Option(False, "--verbose/--no-verbose"),
):
    """Check health of all servers in an environment."""
    if verbose:
        typer.echo(f"Checking {env.value} environment in {region}...")
    
    # ... implementation


@app.command()
def restart(
    server_name: str = typer.Argument(..., help="Server to restart"),
    force: bool = typer.Option(False, "--force", help="Skip confirmation"),
):
    """Restart a named server."""
    if not force:
        typer.confirm(f"Restart '{server_name}'?", abort=True)
    typer.echo(f"Restarting {server_name}...")


if __name__ == "__main__":
    app()
```

**When to use each:**
- **argparse**: Already in standard library; fine for simple scripts
- **click**: Excellent for complex CLIs with subcommands, callbacks, and group context
- **typer**: Best when you love type hints and want the least code; built on top of click

---

## Chapter 11 Summary

- `argparse` is built-in and works well for simple CLIs
- `click` uses decorators and is the professional standard for complex CLIs
- `typer` generates CLIs from type hints — minimal boilerplate
- Always define `--help` text; it makes tools self-documenting
- Use `sys.exit(1)` or `raise SystemExit(1)` to indicate failure
- Register CLI commands in `pyproject.toml [project.scripts]` for installation

---

# Chapter 12: Cloud SDKs
## boto3 (AWS), google-cloud, azure-sdk

---

## Python as Your Cloud Control Plane

The cloud providers all offer official Python SDKs. These let you do everything you can do in the AWS Console, GCP Dashboard, or Azure Portal — but programmatically, repeatedly, and at scale.

In this chapter, we'll focus primarily on AWS's `boto3` (the most widely used), with an overview of GCP and Azure patterns.

---

## boto3 — The AWS SDK for Python

```bash
pip install boto3
```

### Authentication

`boto3` looks for credentials in this order:
1. Passed directly to the client (not recommended)
2. Environment variables: `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN`
3. `~/.aws/credentials` file
4. AWS IAM instance profile (when running on EC2)
5. IAM roles for ECS tasks or Lambda functions

```python
import boto3
import os

# Option 1: Use the default credential chain (recommended — works everywhere)
ec2 = boto3.client("ec2", region_name="us-east-1")

# Option 2: Specific profile from ~/.aws/credentials
session = boto3.Session(profile_name="prod-account")
ec2 = session.client("ec2", region_name="us-east-1")

# Option 3: Environment variables (set before running script)
# export AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE
# export AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
# export AWS_DEFAULT_REGION=us-east-1
ec2 = boto3.client("ec2")  # Picks up from environment
```

### Clients vs Resources

boto3 has two interfaces:
- **Client** — low-level, maps directly to AWS API; returns raw dicts
- **Resource** — high-level, object-oriented; more Pythonic but fewer features

```python
# Client interface
ec2_client = boto3.client("ec2", region_name="us-east-1")
response = ec2_client.describe_instances()
# response is a raw dictionary with everything AWS returned

# Resource interface
ec2_resource = boto3.resource("ec2", region_name="us-east-1")
instances = list(ec2_resource.instances.all())
# instances is a list of Instance objects with .id, .state, .tags, etc.
```

### Listing EC2 Instances

```python
import boto3
from typing import List, Dict, Any

def get_all_instances(region: str) -> List[Dict[str, Any]]:
    """Get all EC2 instances in a region with their key details."""
    ec2 = boto3.client("ec2", region_name=region)
    
    instances = []
    
    # AWS returns results in "pages" — DescribeInstances can return many instances
    paginator = ec2.get_paginator("describe_instances")
    
    for page in paginator.paginate():
        # Reservations group instances (an artifact of the old EC2-Classic era)
        for reservation in page["Reservations"]:
            for instance in reservation["Instances"]:
                # Extract tags into a simple dict
                tags = {
                    tag["Key"]: tag["Value"]
                    for tag in instance.get("Tags", [])
                }
                
                instances.append({
                    "id": instance["InstanceId"],
                    "type": instance["InstanceType"],
                    "state": instance["State"]["Name"],  # pending, running, stopped, terminated
                    "region": region,
                    "availability_zone": instance["Placement"]["AvailabilityZone"],
                    "private_ip": instance.get("PrivateIpAddress"),
                    "public_ip": instance.get("PublicIpAddress"),
                    "name": tags.get("Name", "(unnamed)"),
                    "environment": tags.get("env", tags.get("Environment", "unknown")),
                    "launch_time": instance["LaunchTime"].isoformat(),
                    "tags": tags,
                })
    
    return instances


def get_instances_across_regions(regions: List[str]) -> Dict[str, List[Dict]]:
    """Get EC2 instances from multiple regions."""
    all_instances = {}
    
    for region in regions:
        try:
            all_instances[region] = get_all_instances(region)
            print(f"Found {len(all_instances[region])} instances in {region}")
        except Exception as e:
            print(f"Error fetching instances in {region}: {e}")
            all_instances[region] = []
    
    return all_instances


# Usage
regions = ["us-east-1", "us-west-2", "eu-west-1"]
inventory = get_instances_across_regions(regions)

for region, instances in inventory.items():
    running = [i for i in instances if i["state"] == "running"]
    print(f"\n{region}: {len(running)}/{len(instances)} running")
    for inst in running[:5]:  # Show first 5
        print(f"  {inst['id']} | {inst['type']} | {inst['name']}")
```

### S3 Operations

```python
import boto3
import json
from pathlib import Path

s3 = boto3.client("s3")

# List all buckets
response = s3.list_buckets()
for bucket in response["Buckets"]:
    print(f"{bucket['Name']} (created: {bucket['CreationDate']})")

# Upload a file
s3.upload_file(
    Filename="/tmp/report.json",
    Bucket="my-devops-reports",
    Key="reports/2024-01/health-check.json",  # S3 "path"
    ExtraArgs={"ContentType": "application/json"}
)

# Download a file
s3.download_file(
    Bucket="my-devops-reports",
    Key="reports/2024-01/health-check.json",
    Filename="/tmp/downloaded-report.json"
)

# Read a file directly without downloading
response = s3.get_object(Bucket="my-devops-reports", Key="configs/app.json")
config = json.loads(response["Body"].read().decode("utf-8"))

# List objects in a bucket (paginated)
paginator = s3.get_paginator("list_objects_v2")
for page in paginator.paginate(Bucket="my-devops-reports", Prefix="reports/"):
    for obj in page.get("Contents", []):
        print(f"{obj['Key']} ({obj['Size']} bytes)")
```

### Error Handling with boto3

```python
import boto3
from botocore.exceptions import ClientError, NoCredentialsError, EndpointResolutionError

def safe_describe_instance(instance_id: str, region: str):
    """Describe an EC2 instance with proper error handling."""
    ec2 = boto3.client("ec2", region_name=region)
    
    try:
        response = ec2.describe_instances(InstanceIds=[instance_id])
        reservations = response["Reservations"]
        
        if not reservations:
            return None  # Instance not found
        
        return reservations[0]["Instances"][0]
    
    except ClientError as e:
        error_code = e.response["Error"]["Code"]
        error_message = e.response["Error"]["Message"]
        
        if error_code == "InvalidInstanceID.NotFound":
            print(f"Instance {instance_id} does not exist in {region}")
            return None
        elif error_code == "UnauthorizedOperation":
            print(f"No permission to describe instances in {region}")
            raise PermissionError(f"AWS permission denied: {error_message}") from e
        else:
            raise RuntimeError(f"AWS error {error_code}: {error_message}") from e
    
    except NoCredentialsError:
        raise RuntimeError("AWS credentials not configured. Run 'aws configure' or set env vars.")
```

---

## GCP — google-cloud-python

```bash
pip install google-cloud-compute google-cloud-storage
```

```python
from google.cloud import compute_v1
from google.cloud import storage

# Authenticate via: gcloud auth application-default login
# Or set GOOGLE_APPLICATION_CREDENTIALS=/path/to/service-account.json

# List GCE instances
def list_gce_instances(project_id: str, zone: str):
    instance_client = compute_v1.InstancesClient()
    instances = instance_client.list(project=project_id, zone=zone)
    
    for instance in instances:
        print(f"{instance.name}: {instance.status}")

# GCS (Google Cloud Storage)
storage_client = storage.Client()

# List buckets
for bucket in storage_client.list_buckets():
    print(bucket.name)

# Upload to GCS
bucket = storage_client.bucket("my-bucket")
blob = bucket.blob("configs/app.yaml")
blob.upload_from_filename("/tmp/app.yaml")
```

---

## Azure — azure-sdk-for-python

```bash
pip install azure-identity azure-mgmt-compute azure-storage-blob
```

```python
from azure.identity import DefaultAzureCredential
from azure.mgmt.compute import ComputeManagementClient
from azure.storage.blob import BlobServiceClient

# DefaultAzureCredential tries multiple auth methods:
# service principal env vars, managed identity, az CLI credentials, etc.
credential = DefaultAzureCredential()

subscription_id = "your-subscription-id"

# List VMs
compute_client = ComputeManagementClient(credential, subscription_id)
vms = compute_client.virtual_machines.list_all()

for vm in vms:
    print(f"{vm.name} in {vm.location}")

# Azure Blob Storage
blob_service = BlobServiceClient(
    account_url="https://mystorageaccount.blob.core.windows.net",
    credential=credential
)

container = blob_service.get_container_client("mycontainer")
blob_client = container.get_blob_client("config/app.json")
blob_client.upload_blob(json.dumps(config).encode(), overwrite=True)
```

---

## Chapter 12 Summary

- **boto3** is the AWS SDK — use `client` for full API access, `resource` for high-level OO interface
- Always use paginators (`get_paginator`) for AWS API calls that return lists
- Use `ClientError` to handle AWS-specific errors; check `e.response["Error"]["Code"]`
- Never hardcode credentials — use the default credential chain
- **google-cloud** libraries use `DefaultCredentials` (application default credentials)
- **azure-sdk** uses `DefaultAzureCredential` which tries multiple auth methods automatically
- All cloud SDKs follow similar patterns: authenticate → create client → make API calls

---

# Chapter 13: Type Hints, Mypy, and Linting
## black, flake8, pylint, ruff

---

## Why Code Quality Tools Matter

Imagine a team of five engineers all writing Python with different habits — different indentation styles, different quote styles, different naming conventions. Reading each other's code feels like reading five different languages. Code reviews become arguments about formatting instead of logic.

Code quality tools solve this by automating standards.

---

## Type Hints — Making Python Self-Documenting

Python is dynamically typed, but since Python 3.5 you can add **type hints** (also called type annotations). They don't change how the code runs, but they document what types are expected and let tools catch type errors before they reach production:

```python
from typing import Optional, List, Dict, Tuple, Union
from pathlib import Path

# Without type hints — what does this function accept? What does it return?
def check_server(hostname, port, timeout):
    pass

# With type hints — immediately clear
def check_server(
    hostname: str,
    port: int,
    timeout: float = 10.0
) -> dict:
    pass

# More complex types
def get_instances(
    regions: List[str],
    filters: Optional[Dict[str, str]] = None
) -> Dict[str, List[dict]]:
    pass

# Union — accepts multiple types
def parse_config(source: Union[str, Path]) -> dict:
    pass

# Optional is shorthand for Union[X, None]
def find_server(name: str) -> Optional[dict]:
    # Returns a dict if found, or None if not found
    pass
```

### Python 3.10+ Simplified Syntax

Python 3.10 introduced cleaner syntax for some common cases:

```python
# 3.10+: Use | instead of Union
def parse_config(source: str | Path) -> dict:
    pass

# 3.10+: X | None instead of Optional[X]
def find_server(name: str) -> dict | None:
    pass

# 3.9+: Use built-in types directly (no need to import from typing)
def get_instances(regions: list[str]) -> dict[str, list[dict]]:
    pass
```

### Dataclasses with Types

Type hints shine with dataclasses (from Chapter 2):

```python
from dataclasses import dataclass, field
from typing import Optional, List

@dataclass
class ServerConfig:
    name: str
    ip_address: str
    region: str
    port: int = 443
    tags: dict[str, str] = field(default_factory=dict)
    health_path: str = "/health"
    ssl_cert_path: Optional[str] = None
```

---

## mypy — Static Type Checking

`mypy` reads your type hints and checks for type errors without running your code:

```bash
pip install mypy
mypy myproject/          # Check all Python files in myproject/
mypy src/                # Check src/ directory
mypy --strict script.py  # Stricter checking
```

Example of what mypy catches:

```python
def add_numbers(a: int, b: int) -> int:
    return a + b

result = add_numbers("10", 20)  # mypy error: Argument 1 has type "str"; expected "int"
result = add_numbers(10, 20)    # Fine
doubled: str = result           # mypy error: Incompatible types in assignment
```

Configure mypy in `pyproject.toml`:

```toml
[tool.mypy]
python_version = "3.11"
warn_return_any = true
warn_unused_ignores = true
disallow_untyped_defs = true  # Require type hints on all functions
ignore_missing_imports = true  # Don't error on untyped third-party libraries
```

---

## Formatting with black

`black` is an opinionated code formatter — it has almost no configuration, and it reformats your code to look consistent. Using `black` eliminates all style debates:

```bash
pip install black

# Format a file (modifies in place)
black my_script.py

# Format all Python files in a directory
black src/

# Check if files would change (don't modify — use in CI)
black --check src/

# Show what would change
black --diff src/
```

Example of what black does:

```python
# Before black
x={"key":"value","another_key":"another_value","third":True}
def   foo( x,y,z ):
    return   x+y+z

# After black
x = {"key": "value", "another_key": "another_value", "third": True}

def foo(x, y, z):
    return x + y + z
```

black enforces: 88-character line length by default, double quotes, consistent spacing, trailing commas.

---

## ruff — The Fast Modern Linter

`ruff` is a extremely fast Python linter written in Rust that replaces `flake8`, `isort`, and many `pylint` checks in a single tool:

```bash
pip install ruff

# Lint a file
ruff check my_script.py

# Lint with automatic fixes for fixable issues
ruff check --fix src/

# Format (ruff also has a formatter similar to black)
ruff format src/
```

Example output:
```
src/health_checker.py:10:5: F401 [*] `os` imported but unused
src/health_checker.py:45:9: E501 Line too long (92 > 88 characters)
src/health_checker.py:67:1: I001 [*] Import block is un-sorted or un-formatted
```

Configure ruff in `pyproject.toml`:

```toml
[tool.ruff]
line-length = 88
target-version = "py311"

[tool.ruff.lint]
select = [
    "E",   # pycodestyle errors
    "W",   # pycodestyle warnings
    "F",   # pyflakes (unused imports, undefined names, etc.)
    "I",   # isort (import sorting)
    "N",   # pep8-naming conventions
    "UP",  # pyupgrade (modernise old Python syntax)
    "B",   # flake8-bugbear (common bugs and design issues)
]
ignore = [
    "E501",  # Line too long — handled by formatter
]
```

---

## Putting It All Together: Pre-commit Hooks

Pre-commit hooks run linters and formatters automatically before every git commit:

```bash
pip install pre-commit
```

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.1.6
    hooks:
      - id: ruff
        args: [--fix]
      - id: ruff-format

  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.7.1
    hooks:
      - id: mypy
        additional_dependencies: [types-requests, types-pyyaml]
```

```bash
# Install the hooks
pre-commit install

# Now every git commit runs ruff and mypy automatically
# To run manually on all files
pre-commit run --all-files
```

---

## Chapter 13 Summary

- **Type hints** document expected types and enable static analysis — use them on all function signatures
- **mypy** checks type hints statically — catches bugs before runtime
- **black** formats code automatically — eliminates style debates
- **ruff** is a fast, modern linter that covers what `flake8`, `isort`, and many `pylint` rules do
- Configure all tools in `pyproject.toml` for consistency
- Use **pre-commit** hooks to enforce standards automatically on every commit

---

# Chapter 14: Testing in Python
## pytest, unittest, mocking, fixtures

---

## Why Tests Are Not Optional in DevOps Scripts

Imagine your "list EC2 instances" script has a bug that crashes when it encounters a terminated instance. You don't notice because terminated instances are rare in dev. Then you run it against production: 1,200 instances, 47 of them terminated — script crashes, report incomplete, on-call engineer woken at 3am.

Tests catch these scenarios before they reach production. In DevOps tooling, where scripts often run unattended and trigger downstream automation, bugs are expensive.

---

## pytest — The Standard Testing Framework

Install: `pip install pytest pytest-cov`

### Your First Test

```python
# src/health_checker.py
def is_valid_port(port: int) -> bool:
    """Check if a port number is valid (1-65535)."""
    return isinstance(port, int) and 1 <= port <= 65535

def format_server_name(name: str, env: str, region: str) -> str:
    """Format a server name: name-env-region."""
    return f"{name}-{env}-{region}".lower()
```

```python
# tests/test_health_checker.py
import pytest
from health_checker import is_valid_port, format_server_name

# Test functions MUST start with 'test_'
def test_valid_port_returns_true():
    assert is_valid_port(80) is True
    assert is_valid_port(443) is True
    assert is_valid_port(65535) is True

def test_invalid_port_returns_false():
    assert is_valid_port(0) is False       # Below minimum
    assert is_valid_port(65536) is False   # Above maximum
    assert is_valid_port(-1) is False      # Negative

def test_port_type_check():
    assert is_valid_port("80") is False    # String, not int
    assert is_valid_port(80.5) is False    # Float, not int

def test_format_server_name():
    result = format_server_name("web", "PROD", "US-EAST-1")
    assert result == "web-prod-us-east-1"

def test_format_server_name_already_lowercase():
    result = format_server_name("db", "dev", "eu-west-1")
    assert result == "db-dev-eu-west-1"
```

```bash
# Run all tests
pytest

# Run with verbose output
pytest -v

# Run a specific file
pytest tests/test_health_checker.py

# Run a specific test
pytest tests/test_health_checker.py::test_valid_port_returns_true

# Show test coverage
pytest --cov=src --cov-report=term-missing
```

---

## Fixtures — Reusable Test Setup

A **fixture** is a function that provides data or objects to tests. It runs before the test, and can clean up after:

```python
import pytest
import json
import tempfile
from pathlib import Path

# Simple fixture — returns a value
@pytest.fixture
def sample_servers():
    """Returns a list of sample server dicts for testing."""
    return [
        {"name": "web-01", "ip": "10.0.1.10", "port": 443, "healthy": True},
        {"name": "web-02", "ip": "10.0.1.11", "port": 443, "healthy": True},
        {"name": "db-01",  "ip": "10.0.2.10", "port": 5432, "healthy": False},
    ]

# Fixture with cleanup (using yield)
@pytest.fixture
def temp_config_file():
    """Creates a temporary config file and cleans it up after the test."""
    config = {
        "environment": "test",
        "database": {"host": "localhost", "port": 5432}
    }
    
    # Setup — runs before the test
    with tempfile.NamedTemporaryFile(mode="w", suffix=".json", delete=False) as f:
        json.dump(config, f)
        temp_path = Path(f.name)
    
    yield temp_path  # The test receives this value
    
    # Teardown — runs after the test (even if the test fails)
    temp_path.unlink(missing_ok=True)

# Fixtures can use other fixtures
@pytest.fixture
def loaded_config(temp_config_file):
    """Loads the config from the temp file."""
    with open(temp_config_file) as f:
        return json.load(f)


# Using fixtures in tests
def test_server_count(sample_servers):
    assert len(sample_servers) == 3

def test_unhealthy_servers(sample_servers):
    unhealthy = [s for s in sample_servers if not s["healthy"]]
    assert len(unhealthy) == 1
    assert unhealthy[0]["name"] == "db-01"

def test_config_loads(loaded_config):
    assert loaded_config["environment"] == "test"
    assert loaded_config["database"]["port"] == 5432
```

---

## Parametrize — Testing Multiple Cases Cleanly

Instead of writing 10 near-identical test functions, use `@pytest.mark.parametrize`:

```python
import pytest
from health_checker import is_valid_port

@pytest.mark.parametrize("port,expected", [
    (1, True),        # Minimum valid port
    (80, True),       # Common HTTP
    (443, True),      # Common HTTPS
    (65535, True),    # Maximum valid port
    (0, False),       # Below minimum
    (65536, False),   # Above maximum
    (-1, False),      # Negative
    ("80", False),    # String instead of int
    (None, False),    # None
    (80.5, False),    # Float
])
def test_is_valid_port(port, expected):
    assert is_valid_port(port) == expected
```

This runs the test 10 times, once for each set of parameters. pytest shows you which cases pass and fail individually.

---

## Mocking — Testing Without Real External Dependencies

When your code makes API calls, reads files, or calls external services, tests shouldn't actually do those things. Tests should be fast, isolated, and not depend on network connectivity or AWS credentials.

**Mocking** replaces real functions/objects with fake ones that return predictable results.

```python
# src/aws_client.py
import boto3

def count_running_instances(region: str) -> int:
    """Count running EC2 instances in a region."""
    ec2 = boto3.client("ec2", region_name=region)
    paginator = ec2.get_paginator("describe_instances")
    
    count = 0
    for page in paginator.paginate(
        Filters=[{"Name": "instance-state-name", "Values": ["running"]}]
    ):
        for reservation in page["Reservations"]:
            count += len(reservation["Instances"])
    
    return count
```

```python
# tests/test_aws_client.py
import pytest
from unittest.mock import patch, MagicMock
from aws_client import count_running_instances

def test_count_running_instances():
    # Create fake AWS response data
    mock_page = {
        "Reservations": [
            {"Instances": [{"InstanceId": "i-001"}, {"InstanceId": "i-002"}]},
            {"Instances": [{"InstanceId": "i-003"}]},
        ]
    }
    
    # Create a mock paginator that returns our fake data
    mock_paginator = MagicMock()
    mock_paginator.paginate.return_value = [mock_page]  # Returns one page
    
    # Create a mock EC2 client
    mock_ec2 = MagicMock()
    mock_ec2.get_paginator.return_value = mock_paginator
    
    # Patch boto3.client to return our mock EC2 client
    with patch("aws_client.boto3.client", return_value=mock_ec2):
        result = count_running_instances("us-east-1")
    
    # Verify the result
    assert result == 3
    
    # Verify the right API calls were made
    mock_ec2.get_paginator.assert_called_once_with("describe_instances")
    mock_paginator.paginate.assert_called_once_with(
        Filters=[{"Name": "instance-state-name", "Values": ["running"]}]
    )

def test_count_running_instances_empty_region():
    """Test behavior when no instances exist."""
    mock_paginator = MagicMock()
    mock_paginator.paginate.return_value = [{"Reservations": []}]
    
    mock_ec2 = MagicMock()
    mock_ec2.get_paginator.return_value = mock_paginator
    
    with patch("aws_client.boto3.client", return_value=mock_ec2):
        result = count_running_instances("eu-west-1")
    
    assert result == 0
```

### Using pytest-mock (Cleaner Mocking)

```bash
pip install pytest-mock
```

```python
def test_count_instances_with_mocker(mocker):
    """pytest-mock provides 'mocker' fixture — cleaner than unittest.mock"""
    mock_paginator = mocker.MagicMock()
    mock_paginator.paginate.return_value = [
        {"Reservations": [{"Instances": [{"InstanceId": "i-001"}]}]}
    ]
    
    mock_ec2 = mocker.MagicMock()
    mock_ec2.get_paginator.return_value = mock_paginator
    
    mocker.patch("aws_client.boto3.client", return_value=mock_ec2)
    
    result = count_running_instances("us-east-1")
    assert result == 1
```

---

## Testing Your CLI Tool

```python
from click.testing import CliRunner
from my_cli import cli  # Your click app

def test_check_command_table_output():
    runner = CliRunner()
    
    # Invoke the CLI command
    result = runner.invoke(cli, ["check", "--env", "prod", "--format", "json"])
    
    assert result.exit_code == 0
    output = json.loads(result.output)
    assert isinstance(output, list)

def test_check_command_invalid_env():
    runner = CliRunner()
    result = runner.invoke(cli, ["check", "--env", "invalid"])
    assert result.exit_code != 0
    assert "Invalid value" in result.output
```

---

## Chapter 14 Summary

- **pytest** is the standard Python testing framework — functions starting with `test_` are auto-discovered
- **Fixtures** (decorated with `@pytest.fixture`) provide reusable setup and teardown
- `@pytest.mark.parametrize` runs one test with multiple input/output combinations
- **Mocking** (`unittest.mock.patch`) replaces real dependencies with controllable fakes
- Always mock external dependencies (AWS, HTTP calls, database) in unit tests
- Run `pytest --cov=src --cov-report=term-missing` to measure code coverage
- Use `click.testing.CliRunner` to test click CLI tools

---

# Chapter 15: Concurrency
## threading, asyncio, concurrent.futures

---

## Why Your Scripts Are Slow

You have 50 servers to health-check. Checking each server takes 0.5 seconds. Doing them one at a time (sequentially) takes 25 seconds. Doing them all at once (concurrently) takes about 0.5 seconds — the time of the slowest single check.

This is the value of concurrency. Most DevOps tasks are **I/O-bound** — they spend time waiting for network responses, not computing. While one request waits, your program can send another.

---

## Understanding the Types of Concurrency

Python offers three concurrency models:

1. **threading** — Multiple threads, runs on one CPU core (due to the GIL). Good for I/O-bound tasks.
2. **asyncio** — Single thread, event loop. Best for I/O-bound tasks with high concurrency.
3. **concurrent.futures** — High-level interface wrapping both threads and processes.

For DevOps work (HTTP requests, AWS API calls, SSH commands), **`concurrent.futures`** and **`asyncio`** are the most useful.

---

## concurrent.futures — The High-Level Approach

`concurrent.futures` provides `ThreadPoolExecutor` (threads) and `ProcessPoolExecutor` (processes):

```python
from concurrent.futures import ThreadPoolExecutor, as_completed
import requests
import time

def check_url(url: str, timeout: int = 10) -> dict:
    """Check if a URL is healthy. Returns a result dict."""
    start = time.time()
    try:
        response = requests.get(url, timeout=timeout)
        return {
            "url": url,
            "status_code": response.status_code,
            "healthy": response.ok,
            "latency_ms": round((time.time() - start) * 1000),
            "error": None
        }
    except requests.exceptions.Timeout:
        return {"url": url, "healthy": False, "latency_ms": None, "error": "timeout"}
    except requests.exceptions.ConnectionError as e:
        return {"url": url, "healthy": False, "latency_ms": None, "error": str(e)}

urls_to_check = [
    "https://google.com",
    "https://github.com",
    "https://aws.amazon.com",
    "https://httpbin.org/delay/1",   # Deliberately slow
    "https://httpbin.org/status/500",  # Deliberately returns 500
]

# Sequential — slow
start = time.time()
sequential_results = [check_url(url) for url in urls_to_check]
print(f"Sequential: {time.time() - start:.1f}s")

# Concurrent with ThreadPoolExecutor
start = time.time()
concurrent_results = []

with ThreadPoolExecutor(max_workers=10) as executor:
    # Submit all tasks at once — returns Future objects immediately
    future_to_url = {
        executor.submit(check_url, url): url
        for url in urls_to_check
    }
    
    # as_completed() yields futures as they finish (not in submission order)
    for future in as_completed(future_to_url):
        url = future_to_url[future]
        try:
            result = future.result()   # Get the return value (or raises exception)
            concurrent_results.append(result)
            status = "OK" if result["healthy"] else "FAIL"
            print(f"[{status}] {url} ({result.get('latency_ms', 'N/A')}ms)")
        except Exception as e:
            print(f"[ERROR] {url}: {e}")

print(f"Concurrent: {time.time() - start:.1f}s")

# executor.map() — simpler syntax when you just want all results
with ThreadPoolExecutor(max_workers=10) as executor:
    results = list(executor.map(check_url, urls_to_check, timeout=30))
    # Note: executor.map preserves ORDER (unlike as_completed)
```

### Checking Multiple AWS Regions Concurrently

```python
from concurrent.futures import ThreadPoolExecutor, as_completed
import boto3

def get_instance_count(region: str) -> dict:
    """Count EC2 instances in a region."""
    try:
        ec2 = boto3.client("ec2", region_name=region)
        paginator = ec2.get_paginator("describe_instances")
        
        counts = {"running": 0, "stopped": 0, "other": 0}
        for page in paginator.paginate():
            for reservation in page["Reservations"]:
                for instance in reservation["Instances"]:
                    state = instance["State"]["Name"]
                    if state == "running":
                        counts["running"] += 1
                    elif state == "stopped":
                        counts["stopped"] += 1
                    else:
                        counts["other"] += 1
        
        return {"region": region, "counts": counts, "error": None}
    
    except Exception as e:
        return {"region": region, "counts": None, "error": str(e)}


regions = [
    "us-east-1", "us-east-2", "us-west-1", "us-west-2",
    "eu-west-1", "eu-west-2", "eu-central-1",
    "ap-southeast-1", "ap-northeast-1",
]

with ThreadPoolExecutor(max_workers=len(regions)) as executor:
    futures = {executor.submit(get_instance_count, r): r for r in regions}
    
    print("Region              Running  Stopped  Other")
    print("-" * 48)
    
    for future in as_completed(futures):
        result = future.result()
        if result["error"]:
            print(f"{result['region']:<20} ERROR: {result['error']}")
        else:
            c = result["counts"]
            print(f"{result['region']:<20} {c['running']:<8} {c['stopped']:<8} {c['other']}")
```

---

## asyncio — Concurrency Without Threads

`asyncio` uses a single-threaded event loop. Instead of threads switching execution, your code voluntarily yields control when it's waiting for I/O. This can handle thousands of concurrent operations with minimal memory overhead.

```python
import asyncio
import httpx
import time

async def check_url_async(client: httpx.AsyncClient, url: str) -> dict:
    """Async version of check_url — uses 'await' for I/O operations."""
    start = time.time()
    try:
        response = await client.get(url)   # 'await' yields control while waiting
        return {
            "url": url,
            "healthy": response.is_success,
            "status_code": response.status_code,
            "latency_ms": round((time.time() - start) * 1000),
        }
    except Exception as e:
        return {"url": url, "healthy": False, "error": str(e), "latency_ms": None}


async def check_all_urls(urls: list[str]) -> list[dict]:
    """Check all URLs concurrently using asyncio."""
    async with httpx.AsyncClient(timeout=10.0) as client:
        # Create all coroutines
        tasks = [check_url_async(client, url) for url in urls]
        
        # Run all concurrently with asyncio.gather
        # return_exceptions=True: don't let one failure cancel the rest
        results = await asyncio.gather(*tasks, return_exceptions=True)
        
        return [r if not isinstance(r, Exception) else {"url": "unknown", "error": str(r)}
                for r in results]


# Entry point for async code
async def main():
    urls = [
        "https://google.com",
        "https://github.com", 
        "https://aws.amazon.com",
    ]
    
    results = await check_all_urls(urls)
    
    for result in results:
        status = "OK" if result.get("healthy") else "FAIL"
        print(f"[{status}] {result['url']} ({result.get('latency_ms', 'N/A')}ms)")


# Run the async main function
asyncio.run(main())  # Python 3.7+
```

**Key async/await concepts:**
- `async def` defines a coroutine — a function that can be paused and resumed
- `await` pauses the current coroutine and returns control to the event loop
- `asyncio.gather(*tasks)` runs multiple coroutines concurrently
- `asyncio.run(main())` creates an event loop, runs the main coroutine, and cleans up

---

## When to Use What

| Scenario | Use |
|----------|-----|
| Making many HTTP requests concurrently | `asyncio` + `httpx` or `ThreadPoolExecutor` |
| Calling AWS APIs for multiple regions | `ThreadPoolExecutor` |
| Running shell commands in parallel | `ThreadPoolExecutor` |
| CPU-intensive computation in parallel | `ProcessPoolExecutor` |
| Simple fire-and-forget tasks | `threading.Thread` |

For DevOps work, `ThreadPoolExecutor` is the pragmatic default — it works with all existing (non-async) libraries including `requests` and `boto3`.

---

## Common Beginner Mistakes

**Mistake 1: Forgetting that threads share state**

```python
results = []

def check_and_append(url):
    result = check_url(url)
    results.append(result)  # Dangerous! List.append is thread-safe in CPython, but...
    # If you're doing: results = results + [result]  — NOT thread-safe

# Use lock for genuinely shared mutable state
import threading
lock = threading.Lock()

def check_and_append_safe(url):
    result = check_url(url)
    with lock:
        results.append(result)
```

**Mistake 2: Mixing async and sync code incorrectly**

```python
# WRONG — can't call await outside an async function
result = await check_url_async(client, url)  # SyntaxError

# WRONG — can't call asyncio.run() inside an already-running event loop
async def bad_example():
    asyncio.run(other_coroutine())  # RuntimeError

# CORRECT — use asyncio.run() at the top level, await inside async functions
asyncio.run(main())  # Top level entry point
```

---

## Chapter 15 Summary

- **I/O-bound** tasks (network requests, file reads) benefit from concurrency
- **`ThreadPoolExecutor`** from `concurrent.futures` is the pragmatic choice for most DevOps scripts
- `executor.submit()` returns a `Future`; `as_completed()` yields them as they finish
- **`asyncio`** + `await` is more efficient for very high concurrency (100s of requests)
- `async def` defines a coroutine; `await` pauses it; `asyncio.gather()` runs many concurrently
- Use `ThreadPoolExecutor` with `requests`/`boto3`; use `asyncio` + `httpx` for fully async code
- Always set `max_workers` consciously — too many threads can overwhelm the target system