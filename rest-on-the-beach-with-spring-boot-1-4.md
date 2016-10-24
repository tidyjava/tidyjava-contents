---
id: ghost-30
title: REST on the beach with Spring Boot 1.4
author: Grzegorz Ziemonski
summary: I recently had a chance to play with Spring Boot 1.4 at work. Part of the application I was implementing involved communicating with REST services. I wrote some simple code and I liked it so much, that I decided to share it with you. Since I don't want to bore you with lot's of code and little comment, I made up a story to make things more plausible.
date: 2016-09-23
tags:
    - java
    - spring
---
I recently had a chance to play with Spring Boot 1.4 at work. Part of the application I was implementing involved communicating with REST services. I wrote some simple code and I liked it so much, that I decided to share it with you. Since I don't want to bore you with lot's of code and little comment, I made up a story to make things more plausible.

# Story
Bob, one of the greatest cleaners of planet JEarth needs our help with his day-planning application. Currently, it allows him only to clean code and learn. To be able to put resting on the beach on his schedule, he needs to make sure weather outside is good enough. He won't risk looking outside the window, because he might lose precious milliseconds of coding. We will save him by integrating his application with an imaginary REST API prepared by weather expert Eric (actually, Eric is an expert in every domain).

The application is open-source and is organized around use cases. The use case we're interested in is "Plan for today". Based on the state of the codebase it tells Bob if he can learn new things or he has to take care of the code and clean it. Look, here's `PlanForTodayUseCase`:

```java
@Service
public class PlanForTodayUseCase {
    private final CodeRepository codeRepository;

    public PlanForTodayUseCase(CodeRepository codeRepository) {
        this.codeRepository = codeRepository;
    }

    public Activity get() {
        Code code = codeRepository.get();
        return code.isMessy() ? Activity.CLEAN_CODE : Activity.LEARN;
    }
}
```

Bob gets mad when bugs occur - the whole world stops until it's found and fixed. To prevent stopping the world too much, Bob ensures that his application is always 100% tested. Here's the test for our use case:

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class PlanForTodayUseCaseIntegrationTest {

    @Autowired
    private CodeRepository codeRepository;
    @Autowired
    private PlanForTodayUseCase planForTodayUseCase;

    @Test
    public void shouldBeCleaningWhenCodeIsMessy() throws Exception {
        codeRepository.store(Code.totalMess());

        assertEquals(Activity.CLEAN_CODE, planForTodayUseCase.get());
    }

    @Test
    public void shouldBeLearningWhenCodeIsClean() throws Exception {
        codeRepository.store(Code.beauty());

        assertEquals(Activity.LEARN, planForTodayUseCase.get());
    }
}
```

Integration test like this one is a good way to test classes like `PlanForTodayUseCase`, which would normally rely on heavy mocking and break with the slightest change in any of it's collaborators. Such test is very easy to set up in Spring Boot 1.4 - all we need is `@RunWith(SpringRunner.class)` and `@SpringBootTest`, and everything is automagically set up for us.

# Implementing a REST gateway

Let's get back to Bob's problem. When the code is clean and the weather is good, going to the beach is a better idea than learning. Therefore, we have to be able to extend the use case with a weather check. We'll do that by integrating with (imaginary) Weather API. Let's look at the endpoint it exposes:

```
GET /weather

{
    "temperature": number,
    "rain": boolean
}
```

After a quick chat with Bob, we learn that "good weather", actually means temperature over 20 degrees. He doesn't care whether it rains or not - he always sits under an umbrella (of unit tests). Seems we're ready to implement!

To consume the REST endpoint, we'll use Spring's `RestTemplate`. Spring doesn't ship it as a bean by default, so we'll create a relevant piece of context configuration by hand:

```java
@Configuration
public class GatewaysConfig {
    
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

To hide complexities of communicating with external services, contracts and all unimportant stuff, we'll expose a shielding interface:

```java
public interface WeatherGateway {

    int getTemperature();
}
```

In Clean Architecture *gateway* is used for classes communicating with the database. In other sources, communication with database is done via *repositories*, while *gateways* are usually used for REST clients associated with specific services. We'll follow the second convention.

Having the interface, we can now implement it as a client for Weather API:

```java
@Component
public class WeatherGatewayImpl implements WeatherGateway {
    private final RestTemplate restTemplate;
    private final String weatherApiUrl;

    public WeatherGatewayImpl(RestTemplate restTemplate, @Value("${weather.api.url}") String weatherApiUrl) {
        this.restTemplate = restTemplate;
        this.weatherApiUrl = weatherApiUrl;
    }

    @Override
    public int getTemperature() {
        return restTemplate.getForObject(weatherApiUrl + "/weather", Weather.class).getTemperature();
    }
}
```

Please note that the gateway doesn't return a whole `Weather` class object. Since the clients of the class are not interested whether it rains or not, we're hiding that fact from them, making the interface simpler. This is something most people forget about - we want to encourage *loose coupling*; therefore, we should expose as little detail as possible!

# Testing the gateway

We don't want to make Bob mad, so we better test the client. Testing REST gateways is tricky. A unit test doesn't prove that the communication will be successful, while integration test alone can't always prove that the gateway produces good results. This is why we'll implement both! Ladies and gentleman, the unit test:

```java
public class WeatherGatewayImplTest {
    private static final String BASE_URL = "http://endpoint";
    private static final String WEATHER_ENDPOINT = "http://endpoint/weather";
    private static final int TEMPERATURE = 82;

    private RestTemplate restTemplate = mock(RestTemplate.class);
    private WeatherGateway weatherGateway = new WeatherGatewayImpl(restTemplate, BASE_URL);

    @Before
    public void setUp() throws Exception {
        given(restTemplate.getForObject(WEATHER_ENDPOINT, Weather.class)).willReturn(new Weather(TEMPERATURE, true));
    }

    @Test
    public void shouldReturnCorrectTemperature() throws Exception {
        assertEquals(TEMPERATURE, weatherGateway.getTemperature());
    }

    @Test
    public void shouldQueryCorrectEndpoint() throws Exception {
        weatherGateway.getTemperature();
        verify(restTemplate, times(1)).getForObject(WEATHER_ENDPOINT, Weather.class);
    }
}
```

We tested the gateway both for good results and querying the correct endpoint. This wasn't a complete necessity in this case, but it often is, when endpoint URLs are more complex or have multiple different variations.

The integration test is very simple:

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class WeatherGatewayIntegrationTest {

    @Autowired
    private WeatherGateway weatherGateway;

    @Test
    public void shouldBeSuccessful() throws Exception {
        weatherGateway.getTemperature();
    }
}
```

Sad fact is: we cannot predict what's the weather now, so we can't assert the final value in the integration test. But this test gives us some confidence anyway, because we know that the call was successful - `RestTemplate` throws an exception for unsuccessful calls.

Having tested the gateway so thoroughly we will be able to safely mock it out later without worrying that things won't work together.

# Using the gateway
Now we're ready to make Bob happy by using our gateway in the use case logic. As pointed out before, we have to make sure that code is clean and temperature is over 20 degrees. Here it comes:

```java
@Service
public class PlanForTodayUseCase {
    private final CodeRepository codeRepository;
    private final WeatherGateway weatherGateway;

    public PlanForTodayUseCase(CodeRepository codeRepository, WeatherGateway weatherGateway) {
        this.codeRepository = codeRepository;
        this.weatherGateway = weatherGateway;
    }

    public Activity get() {
        Code code = codeRepository.get();

        if (code.isMessy()) {
            return Activity.CLEAN_CODE;
        } else if (itsWarmEnoughForBob()) {
            return Activity.REST_ON_THE_BEACH;
        } else {
            return Activity.LEARN;
        }
    }

    private boolean itsWarmEnoughForBob() {
        return weatherGateway.getTemperature() > 20;
    }
}
```

And again.. we want things to be well tested. Here's updated test for the use case:

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class PlanForTodayUseCaseIntegrationTest {

    @Autowired
    private CodeRepository codeRepository;
    @MockBean
    private WeatherGateway weatherGateway;
    @Autowired
    private PlanForTodayUseCase planForTodayUseCase;

    @Test
    public void shouldBeCleaningWhenCodeIsMessy() throws Exception {
        codeRepository.store(Code.totalMess());

        assertEquals(Activity.CLEAN_CODE, planForTodayUseCase.get());
    }

    @Test
    public void shouldBeRestingWhenCodeIsCleanAndItsWarm() throws Exception {
        codeRepository.store(Code.beauty());
        given(weatherGateway.getTemperature()).willReturn(21);

        assertEquals(Activity.REST_ON_THE_BEACH, planForTodayUseCase.get());
    }

    @Test
    public void shouldBeLearningWhenCodeIsCleanAndItsCold() throws Exception {
        codeRepository.store(Code.beauty());
        given(weatherGateway.getTemperature()).willReturn(20);

        assertEquals(Activity.LEARN, planForTodayUseCase.get());
    }
}
```

We used `@MockBean` annotation to mock the dependency on `WeatherGateway`, which we tested left and right already. This makes the test easier to write and manage.

# Summary
Spring Boot provides everything you need to implement and test simple REST integrations. We start off the implementation with a shielding interface, that specifies what we really need from the external service. Then we implement the interface using Spring's `RestTemplate`. We test the implementation using both unit and integration test to make sure that we're hitting the right endpoint, returning the right value and the communication is successful. Thanks to wonderful clean-up in 1.4, all we need to set up an integration test is two annotations - `@RunWith(SpringRunner.class)` and `@SpringBootTest`. Finally, we mock out the gateway in the use case classes using `@MockBean`, so that our functional tests are not impacted by any external party. Last but not least: remember to test your classes well or Bob will get mad! :)

All code used in the examples is available here:
https://github.com/tidyjava/bobs-calendar