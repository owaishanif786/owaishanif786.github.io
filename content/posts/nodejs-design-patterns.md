---
title: "Nodejs Design Patterns"
date: 2022-08-06T02:17:08+05:00
draft: true
---

# Node.js Design Patterns

## Singleton Pattern

Used when we need only one instance of class regardless of how many times we call it. example logger, database connection
simply to keep the instance as singleton, export with new keyword so that node.js will cache the object.

```javascript
class Database{
    constructor(){
        
    }
}
module.exports = new Database()
```

The other long way to create another class and export it with another class by check Singleton.instance = new Logger()


