## Start Networking
Let's start adding some data. The Api Documentation can be found [here](https://onboarding-todo-api.development.appwi.se/api/v1/docs/).
username: `appwise`
password: `password`

This is the endpoint that will be used: `https://onboarding-todo-api.development.appwi.se`

We use [Room](https://developer.android.com/jetpack/androidx/releases/room) for our local database. Room is an abstraction layer over SQLite.
It makes it easier to work with SQLite. We use [Retrofit](https://square.github.io/retrofit/) for our network calls.
Retrofit is a type-safe HTTP client for Android and Java.

### 11.1 Setup Networking

#### 11.1.1 Init Database
Create a new package called **com.wiselab.<<name>>.data**. This is where we will put all our data related classes.
In this package create a new package called **com.wiselab.<<name>>.data.database**.

* Create a new class called **AppDatabase** in the **database** package. This class will be used to create our local
  database. Add the following code to the **Database** class:

```kotlin
@Database(
    entities = [],
    version = 1,
    exportSchema = false
)
abstract class AppDatabase : RoomDatabase() {

    companion object {
        const val DATABASE_NAME = "todoApp.db"
    }
}
```

#### 11.1.3 Add product flavours
* Open the **build.gradle** file of the **app** module.
* 
* Add the following code to the **android** block:

```kotlin
flavorDimensions += "env"
productFlavors {
    create("dev") {
        buildConfigField("String", "API_HOST", "\"https://onboarding-todo-api.development.appwi.se/\"")
        buildConfigField("String", "API_CLIENT_ID", "\"\"")
        buildConfigField("String", "API_CLIENT_SECRET", "\"\"")

        dimension = "env"
        applicationIdSuffix = ".dev"
        versionNameSuffix = "-dev"
    }
}
```

* Client ID: `bdba526c-31b3-4740-a4e6-bfbaf96ec62e`
* Client Secret: `55d5f96e-eb16-4e98-8822-27cba3474e01`

The **product flavours** are used to create different versions of the app. We will use this to create a development version of the app.
The development version will use the development API. In production apps we also include versions for staging and production. This way we can test the app on different environments.
Make sure to **build** the project after adding the product flavours. This way the **BuildConfig** will be generated.

#### 11.1.4 Initialise the core Networking module
* In the app class, add a new function called `initNetworking()`:

```kotlin
private fun initNetworking() {
    Networking.Builder(this)
        .setPackageName(packageName)
        .setAppName(getString(R.string.app_name))
        .setClientIdValue(BuildConfig.API_CLIENT_ID)
        .setClientSecretValue(BuildConfig.API_CLIENT_SECRET)
        .setApiVersion("1").apply {
            if (isProxymanEnabled()) {
                registerProxymanService(context = this@App)
            }
        }.build()
}
```
This is where we initialise the Networking module. We use the `Networking.Builder` to create a new instance of the `Networking` class.
Call this function from the `onCreate()` function of the app class.

### 11.2 Create the RestClients
* Create a new package called **networking** like this: **com.wiselab.<<name>>.networking**
* In this package create 2 new classes called **ProtectedClient** and **UnProtectedClient**.
* For now create them like this:
    
```kotlin
class ProtectedClient(private val unprotectedClient: UnprotectedClient) : BaseRestClient() { 
    override val protectedClient = true
    override fun getBaseUrl() = BuildConfig.API_HOST
    override fun enableProxyManInterceptor() = true
}
```

```kotlin
class UnProtectedClient : BaseRestClient() {
    override val protectedClient = false
    override fun getBaseUrl() = BuildConfig.API_HOST
    override fun enableProxyManInterceptor() = true
}
```

### 11.3 ApiManagerService
We are going to create the **service** that will be used for api token management. This service will be used to get the api token and refresh it when it's expired.

* Create a new package called **service** in the **networking** package like this: **com.wiselab.<<name>>.networking.services**
* In this package create a new interface called **AuthService**:
    
```kotlin
interface AuthService : BaseService {

    //OAuth token
    @FormUrlEncoded
    @POST("/api/oauth/token")
    fun loginWithPassword(
        @Field(NetworkConstants.KEY_GRANT_TYPE) grantType: String,
        @Field(NetworkConstants.KEY_CLIENT_ID) clientId: String,
        @Field(NetworkConstants.KEY_CLIENT_SECRET) client_secret: String,
        @Field("username") username: String,
        @Field("password") code: String,
        @Field("scope") scopeRead: String = "read write"
    ): Call<AccessToken>

    //OAuth Refresh Token
    @FormUrlEncoded
    @POST("/api/oauth/token")
    fun refreshToken(
        @Field(NetworkConstants.KEY_GRANT_TYPE) grantType: String,
        @Field(NetworkConstants.KEY_CLIENT_ID) clientId: String,
        @Field(NetworkConstants.KEY_CLIENT_SECRET) client_secret: String,
        @Field(NetworkConstants.FIELD_IDENTIFIER_REFRESH_TOKEN) refreshToken: String
    ): Call<AccessToken>
}
```
Here we define the endpoints that we will use to get the api token and refresh it when it's expired. The fields that are used in the endpoints are defined in the **NetworkConstants** class.

### 11.4 Repository
We are going to create the **repository** that will be used to get the api token and refresh it when it's expired.

* Create a new package called **repository** in the **data** package like this: **com.wiselab.<<name>>.data.repository**
* In this package create a new **package** called **authRepo**:
* In this package create a new interface called **AuthRepo**:
    
```kotlin
interface AuthRepo : BaseRepository {

    suspend fun login(email: String, password: String)
}
```
We create this interface so we can use it in the **AuthRepoImpl** class. This way we can use **dependency injection** to inject the **AuthRepo** interface where needed.
You also see we make use of a suspend function. This is because we will make **asynchronous** calls to the api. We will use **coroutines** to make these calls.
[Coroutines](https://developer.android.com/kotlin/coroutines) are a Kotlin feature that convert async callbacks for long-running tasks, such as database or network access, into sequential code.
It's a bit more advanced so we won't go into detail here. You can read more about it in the documentation.


* In the **authRepo** package create a new class called **AuthRepoImpl**:
    
```kotlin
class AuthRepoImpl(
    private val db: AppDatabase,
    private val protectedClient: ProtectedClient,
    private val unprotectedClient: UnprotectedClient
) : AuthRepo {

    override suspend fun login(email: String, password: String) {
        val accessToken = doCall(
            unprotectedClient.getService<AuthService>().loginWithPassword(
                "password",
                Networking.getClientIdValue(),
                Networking.getClientSecretValue(),
                email,
                password
            )
        )

        Networking.saveAccessToken(accessToken)
    }
}
```
As parameters we pass the **AppDatabase**, **ProtectedClient** and **UnprotectedClient**. We use the **UnprotectedClient** to get the **AuthService** service. We use this service to get the api token. We use the **ProtectedClient** to save the api token in the **Networking** class.
Make sure to take a look at the `doCall()` function that is used to make the api call. This function is defined in the **BaseRepository** class.

### 11.5 Hawk
Hawk is secure, simple key-value storage for Android. It's encrypted and it's easy to use. We will use it to store some values.

* Create a package **Util** like this: **com.wiselab.<<name>>.util**
* In this package create a new **package** called **managers**
* In this package create a new object called **HawkManager**:
    
```kotlin
object HawkManager {
    private const val CURRENT_USER = "current_user"
    
    var currentUserId: String by HawkValueDelegate(CURRENT_USER, defaultValue = "")
}
```
Here we define the keys that we will use to store values in Hawk. We also define the default values for the values that we store in Hawk.
 
### 11.6 Create a first model
Let's implement the `userInfo` endpoint. This endpoint will return the user info of the logged in user.

* Create a new package called **entity** in the **data** package like this: **com.wiselab.<<name>>.data.entity**
Here we will store our entities. These are the objects that we will use to store the data that we get from the api.
* Check the API documentation to see what data we get from the api. We will create a new data class called **User**:
    
```kotlin
import androidx.room.Entity
import androidx.room.PrimaryKey
import be.appwise.room.BaseEntity
import com.google.gson.annotations.SerializedName
import org.joda.time.DateTime

@Entity(tableName = User.USER_TABLE_NAME)
data class User(
    @PrimaryKey
    @SerializedName("uuid")
    override val id: String,
    val createdAt: DateTime? = DateTime(),
    val updatedAt: DateTime? = DateTime(),
    val email: String? = "",
    val firstName: String? = "",
    val lastName: String? = "",
    val role: UserRole? = UserRole.USER
) : BaseEntity {
    companion object {
        const val USER_TABLE_NAME = "user"
    }
}
```
We will give the `user` class as an example. The entity annotation is used to tell Room that this is an entity. 
* The `tableName` parameter is used to define the name of the table in the database. 
* The `id` parameter is used to define the primary key of the table.
* The `BaseEntity` interface is used to make sure that all entities have an id.
* Don't forget to create the **UserRole** enum class, all the options are visible in the documentation.

Now we have created our first Entity. We will use this entity to store the data that we get from the api.
But first add this entity to the **AppDatabase** class in the annotation:

```kotlin
@Database(
    entities = [
        User::class
    ],
    version = 1,
    exportSchema = false
) {}
```


* In the **AuthService** class add a new function called `userInfo()`
* This is a Get call with the endpoint specified in the annotation, it returns a `User` object:
    
```kotlin
@GET("/api/oauth/userinfo")
    fun userInfo(): Call<User>
```

* Add a new function to the AuthRepo interface called `userInfo()` and implement it in the **AuthRepoImpl** class:
    
```kotlin
override suspend fun userInfo() {
        doCall(protectedClient.getService<AuthService>().userInfo()).also {
            HawkManager.currentUserId = it.id
        }
    }
```

### 11.7 DAO
A dao is used to access the database. We will create a dao for the `User` entity.

* Create a new package called **dao** in the **data** package like this: **com.wiselab.<<name>>.data.dao**
* In this package create a new **abstract** class called **UserDao**:
    
```kotlin
import androidx.room.Dao
import be.appwise.room.BaseRoomDao
import com.wiselab.onboarding_compose.data.entity.User

@Dao
abstract class UserDao : BaseRoomDao<User>(User.USER_TABLE_NAME)
```

* Add this DAO to the **AppDatabase** class, it should look like this:
    
```kotlin
@Database(
    entities = [
        User::class
               ],
    version = 1,
    exportSchema = false
)
abstract class AppDatabase : RoomDatabase() {

    companion object {
        const val DATABASE_NAME = "todoApp.db"
    }
    
    abstract fun userDao(): UserDao
}
```

* Now we want to save our User data we get from the `userInfo()` call into our database. This is done by adding the **insert** line to the `userInfo()` call:

```kotlin
override suspend fun userInfo() {
        doCall(protectedClient.getService<AuthService>().userInfo()).also {
            HawkManager.currentUserId = it.id
            db.userDao().insert(it)
        }
    }
```

### 11.8 Adding login and register
You already added the `login()` function to the **AuthRepo** interface.
Add the `userInfo()` function to the login function like this:

```kotlin
override suspend fun login(email: String, password: String) {
        val accessToken = doCall(
            unprotectedClient.getService<AuthService>().loginWithPassword(
                "password",
                Networking.getClientIdValue(),
                Networking.getClientSecretValue(),
                email,
                password
            )
        )

        Networking.saveAccessToken(accessToken)
        userInfo()
    }
```
This way we will get the user info stored in the database after we logged in.

* Navigate to your `LoginViewModel` and add the `authRepo` as a parameter to the constructor.
* This will be injected so we need to add some code to the appModule:

```kotlin
val appModule = module {
    single {
        Room.databaseBuilder(
            App.instance,
            AppDatabase::class.java,
            AppDatabase.DATABASE_NAME
        )
            .fallbackToDestructiveMigration()
            .build()
    }
    
    single { UnprotectedClient() }
    single { ProtectedClient(get()) }
  
    single<AuthRepo> { AuthRepoImpl(get(), get(), get()) }
  
    viewModel { LoginViewModel() }
}
```
Let's explain what we are doing. We are creating singletons of the `AppDatabase`, `UnprotectedClient`, `ProtectedClient` and `AuthRepoImpl` classes.
This is needed because we want to use the same instance of these classes everywhere in the app. We also want to inject the `AuthRepoImpl` class in the `LoginViewModel`.

* The `AuthRepo` has a dependency on the `AppDatabase`, `UnprotectedClient` and `ProtectedClient` classes. We use the `get()` function to get the instance of these classes.

#### 11.8.1 Login
* In the `LoginViewModel` add a new function called `login()`:

```kotlin
fun login(email: String, password: String) = launchAndLoad { 
    authRepo.login(email, password)
}
```
This function can be called from the onAction of the `LoginViewModel` so we can use it in the `LoginScreen`.
The launchAndLoad function is a function that is defined in the `BaseViewModel`. It's a function that is used to make asynchronous calls.

* We want to add an **extra parameter** to the login function for future purposes.
* At this moment we don't know if the login call **succeeds**. A handy feature of coroutines is that the **coroutine is cancelled** when an exception is thrown.
* Therefore we add the extra parameter `onSuccess` to the `login()` function. This is a function that will be called when the login call succeeds.

```kotlin
fun login(email: String, password: String, onSuccess: () -> Unit) = launchAndLoad {
    authRepo.login(email, password)
    onSuccess()
}
```

This onSuccess function will be used to **navigate** to the main screen when the **login call** succeeds.

#### 11.8.2 Register
Let's create the register functionality. Start by adding a new service called `UserService`
We do this to keep the calls separated like in the **api documentation**.

* Add this to your service: 

```kotlin
@POST("api/v1/users")
fun register(
  @Body userRequest: UserRequest
): Call<User>
```
This is a **@POST** call with the endpoint specified in the annotation, it returns a `User` object as visible in the **documentation**.
The call expects a **body**, also visible in the documentation. You can see this is not the same object as a `User` object.
Therefore we create a new **request** object called `UserRequest`:

* Create a new package called **model** in the **networking** package like this: **com.wiselab.<<name>>.networking.model**
* In this package create a new package called **request**
* In this package create a new data class called **UserRequest** with the data that is necessary for the register call

**Side note**: It's possible you get objects from the api that you **don't** want as an entity. Something that has to be **mapped** to an existing entity for example.
To solve this we create a package called **response** in the **model** package. Here we create the objects that we get from the api. We use these objects to **map** to our entities.

**New Repo**
* Create a new package called **userRepo** in the **repository** package like this: **com.wiselab.<<name>>.data.repository.userRepo**
* You have the knowledge to create this repo with the **register call** yourself. If you need help, you can check the **authRepo**.
* Don't forget to update the appModule with the new repo.
* Make sure to commit and push your code. You may want to create a pull request for this.

Now you are fully set up to create the rest of the application yourself. Use this codelab as a reference if you need help.
Don't be afraid to ask for help if you need it. Good luck! 🚀
