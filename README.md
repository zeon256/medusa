# medusa - simpler jdbc
 [ ![Download](https://api.bintray.com/packages/budinverse/utils/medusa/images/download.svg) ](https://bintray.com/budinverse/utils/medusa/_latestVersion)
 [![MIT Licence](https://badges.frapsoft.com/os/mit/mit.svg?v=103)](https://opensource.org/licenses/mit-license.php)
 
medusa is a jdbc-utilities library that is designed to reduce writing code pertaining to jdbc. No more closing of connection manually, no spawning your own `preparedStatements`. This helps reduce bugs where connection is not clsoed and bugs where column number are wrong.All this in a lightweight library that leverages Kotlin's ability to write DSLs.medusa is not an ORM, it is just a utilities library to help you.

### Gradle
```groovy
repositories {
    jcenter()
}
dependencies {
  compile group: 'mysql', name: 'mysql-connector-java', version: '6.0.6' //depends on the driver you need
  compile 'com.budinverse.utils:Medusa:<latest version>'
}
```
### Config
Setting up your `DatabaseConfig` via properties file
```kotlin
fun main(args: Array<String>) {
    DatabaseConfig.setConfig(configFileDir = "databaseConfig.properties")
}
```

Setting up your `DatabaseConfig` via built-in DSL
```kotlin
fun main(args: Array<String>) {
    dbConfig {
            databaseUser = "root"
            databasePassword = "12345"
            databaseUrl = "jdbc:mysql://localhost/medusa?useLegacyDatetimeCode=false&serverTimezone=UTC"
            driver = "com.mysql.cj.jdbc.Driver"
        }
}
```
## Examples
Assume that all examples has the following `User` class
```kotlin
data class User(val id: Int = 0,
                     val name: String,
                     val age: Int) {
        constructor(resultSet: ResultSet) : this(
                resultSet["id"],
                resultSet["name"],
                resultSet["age"]
        )
    }
```

### Query
A single query
```kotlin
fun getUser() = transaction {
        val user = query<User> {
            //language=MySQL
            statement = "SELECT * FROM User WHERE name = ?"
            values = arrayOf("zeon222")
            type = ::User
        }

     user?.let(::println) // User(id=18, name=Zeon222, age=20)
}
```
Querying list of objects
```kotlin
fun getUser() = transaction {
        val users = query<User> {
            //language=MySQL
            statement = "SELECT * FROM User"
            type = ::User // make sure constructor is available!
        }

     users.let(::println) // [User(id=18, name=Zeon222, age=20), User(id=20, name=Zeon333, age=19)]
}
```

### Insert
```kotlin
fun insertUser(user: User): TransactionResult = transaction {
        exec {
            //language=MySQL
            statement = "INSERT INTO User (name, age) VALUES (?,?)"
            values = arrayOf(user.name, user.age)
        }
    }
    
fun runInsert() {
    val user = User("zeon000", 19)
    val res = insertUser(user)
    
    when(res){
        is Success -> println("Inserted $user!")
        is Fail -> println("Insert $user! failed!")
    }
}    
```

### Update/Delete
For deletion just change the statement
```kotlin
fun insertUpdate(user: User): TransactionResult = transaction {
        exec {
            //language=MySQL
            statement = "UPDATE User SET name = ? WHERE id = ?"
            values = arrayOf(user.name, 1)
        }
    }
    
fun runUpdate() {
    val user = User("zeon000", 19)
    val res = insertUser(user)
    
    when(res){
        is Success -> println("Updated $user!")
        is Fail -> println("Update $user! failed!")
    }
}    
```

### Transaction
Includes usage of `execKeys` which returns `ResultSet?`
```kotlin
fun aTransasction(user: User): TransactionResult {
    var id = 0
    transaction {
                id = insertKeys {
                    //language=MySQL
                    statement = "INSERT INTO User (name, age) VALUES (?,?)"
                    values = arrayOf(user.name, user.age)
                }!![1]
    
                exec {
                    //language=MySQL
                    statement = "UPDATE User SET age = ? WHERE id = ?"
                    values = arrayOf(20, id)
                }
        }
}
    
fun runTransaction() {
    val user = User("zeon000", 19)
    val res = insertUser(user)
    
    when(res){
        is Success -> println("Transaction success!")
        is Fail -> println("Transaction failed!")
    }
}    
```



