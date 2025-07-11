### [< Previous Challenge](./Challenge-02.md) - **[Home](../README.md)** - [Next Challenge >](./Challenge-04.md)

# Challenge 03 - Semantic Kernel Plugins

## Pre-requisites

Completed [Challenge 02](./Challenge-02.md) and have a functional version of the solution running in Visual Studio Code.

## Introduction

Semantic Kernel truly shines in LLM development when you incorporate plugins. These plugins extend the AI's capabilities and provide access to additional knowledge that cannot be built directly into the model through training. Things such as time sensitive data, user specific information, and esoteric knowledge are all areas where the Plugin model can greatly improve the capabilities of your AI. In this challenge, you will implement a time plugin, and a plugin that retrieves the weather for a location to extend the capabilities of your chat bot.

## Description

This challenge will introduce you to building Semantic Kernel Plugins in python, and how to chain plugins using the Auto Function Calling capabilities of Semantic Kernel.

### Challenges

* Launch your AI Chat app, and submit the prompt.

  ```text
  What time is it?
  ```
  
  Since the AI does not have the capability to provide real-time information, you will get a response similar to the following:

  ```text
  I can't provide real-time information, including the current time. You can check the time on your device or through various online sources.
  ```

    Let's fix this by creating a plugin that can provide the current time and other related information.

* **Create a new class in *./plugins* directory for your Time Plugin**. You can reference the [documentation](https://learn.microsoft.com/en-us/semantic-kernel/concepts/plugins/adding-native-plugins?pivots=programming-language-python#defining-a-plugin-using-a-class) for more information on how to create a plugin using a class.
  * Write a time plugin with the following functions:
    1. Return the current Date Time
    1. Return the Year for a date passed in as a parameter
    1. Return the Month for a date passed in as a parameter
    1. Return the Day of Week for a date passed in as a parameter
  * Update the application to add your new Plugin to Semantic Kernel

    :bulb: Use the provided sections in the **chat.py** file to add your new Plugin to Semantic Kernel. Review the documentation [Adding native plugins](https://learn.microsoft.com/en-us/semantic-kernel/concepts/plugins/adding-native-plugins?pivots=programming-language-python) for examples on how to do this.

  * Enable Automatic Function Calling

      In ```chat.py``` below the comment ```// Challenge 03 - Create Prompt Execution Settings``` Configure Semantic Kernel to automatically call the functions in your plugin when the AI recognizes the intent. See [Using Automatic Function Calling](https://learn.microsoft.com/en-us/semantic-kernel/concepts/planning?pivots=programming-language-python#using-automatic-function-calling).

  * Test the AI by launching the application and asking the bot
  
    ```text
    What time is it?
    ```

    Now, the AI should be able to provide the current time by having Semantic Kernel call the ***GetTime*** function in your plugin. The response should be similar to the following:

    ```text
    The current time is 3:43 PM on January 23, 2025.
    ```

    See the [Success Criteria](#success-criteria) section for additional questions to test your Time Plugin.

* **Review the ```geo_coding_plugin.py``` file located in the ***plugins*** directory**
  * Register for a free API key from [Geocoding API](https://geocode.maps.co/) to use in the plugin

  ### Environment Setup

  Now that you've registered for a geocoding API key, update the `.env` file you created in Challenge-02:

  ```
  # Add this to your existing .env file
  GEOCODING_API_KEY="your-geocoding-api-key"
  ```

  * Register the Plugin

  * Run the application and test the Geocoding plugin by submitting the following prompt:

    ```text
    what are the geo-coordinates for Tampa, FL
    ```
  
      The AI should respond with the coordinates for Tampa, similar to the following:
  
      ```text
      The geo-coordinates for Tampa, FL are:

      Latitude: 27.9477595
      Longitude: -82.458444
      ```

* **Create a Plugin that calls a Weather API, and add it to the Semantic Kernel**. You can utilize the Open Meteo API [Here](https://open-meteo.com/en/docs).  

  * Add Methods to your plugin to:
    1. Get the forecast weather at lat/long location for up to 16 days in the future

      ```python
        $"https://api.open-meteo.com/v1/forecast?latitude={latitude}&longitude={longitude}&current=temperature_2m,relative_humidity_2m,apparent_temperature,precipitation,rain,showers,snowfall,weather_code,wind_speed_10m,wind_direction_10m,wind_gusts_10m&hourly=temperature_2m,relative_humidity_2m,apparent_temperature,precipitation_probability,precipitation,rain,showers,snowfall,weather_code,cloud_cover,wind_speed_10m,uv_index&temperature_unit=fahrenheit&wind_speed_unit=mph&precipitation_unit=inch&forecast_days={days}");
       ```

    1. Get the current weather at lat/long location for a number of days in the past

       ```python
        $"https://api.open-meteo.com/v1/forecast?latitude={latitude}&longitude={longitude}&daily=weather_code,temperature_2m_max,temperature_2m_min,apparent_temperature_max,apparent_temperature_min,sunrise,sunset,daylight_duration,uv_index_max,precipitation_sum,rain_sum,showers_sum,snowfall_sum,precipitation_hours,wind_speed_10m_max,wind_gusts_10m_max&temperature_unit=fahrenheit&wind_speed_unit=mph&precipitation_unit=inch&past_days={daysInPast}");
       ```

  * Test your Plugins by asking the following question:
  
    ```text
    What is the weather in San Francisco next Tuesday?
    ```

    :exclamation: The AI should perform the following plan to answer the question but may do so in a different order or different set of functions:

    :one: The AI should ask Semantic Kernel to call the ```GetDate``` function on the Time Plugin to get **today's** date in order to calculate the number of days until **next Thursday**

    :two: Because the Weather Forecast requires a Latitude and Longitude, the AI should instruct Semantic Kernel to call the ```GetLocation``` function on the Geocoding Plugin to get the coordinates for **San Francisco**

    :three: Finally, the AI should ask Semantic Kernel to call the ```GetWeatherForecast``` function on the Weather Plugin passing in the current date/time and Lat/Long to get the weather forecast for **Next Thursday** (expressed as the number of days in the future) at the coordinates for **San Francisco**

    A simplified sequence diagram between Semantic Kernel and AI is shown below:

    ```mermaid
    sequenceDiagram
        participant C as Client
        participant S as Semantic Kernel
        participant A as AI
        C->>S: What is the weather in San Francisco next Tuesday?
        activate C
        S->>+A: What is the weather in San Francisco next Tuesday?
        A-->>-S: Call get_date function
        S->>+A: Results of get_date
        A-->>-S: Call day_of_week function
        S->>+A: Results of day_of_week
        A-->>-S: Call geocode_address function
        S->>+A: Results of geocode_address
        A-->>-S: Call get_weather with lat/long and days in future
        S->>+A: Results of get_weather
        A-->>-S: The weather in San Francisco next Tuesday is...
        S->>C: Here is the weather for San Francisco next Tuesday
        deactivate C
    ```

    :bulb: Set breakpoints in your plugins to verify that the functions are being called correctly and that the data is being passed between the plugins correctly.

## Understanding Semantic Kernel Plugin Architecture

The following diagram illustrates how Semantic Kernel plugins extend AI capabilities through native functions and automatic function calling:

```mermaid
flowchart TB
    subgraph User
        A[User asks about weather in San Francisco]
    end
    
    subgraph SemanticKernel["Semantic Kernel"]
        B[Process user query]
        C{Function choice}
        
        subgraph TimePlugin["Time Plugin"]
            D[GetDate function]
            E[GetDayOfWeek function]
        end
        
        subgraph GeoPlugin["Geo Plugin"]
            F[GetLocation function]
        end
        
        subgraph WeatherPlugin["Weather Plugin"]
            G[GetWeatherForecast function]
        end
        
        H[Combine results & generate response]
    end
    
    subgraph ExternalSystems["External Data Sources"]
        I[Current time data]
        J[Geocoding API]
        K[Weather API]
    end
    
    A -->|user query| B
    B --> C
    
    C -->|"detect: need current date"| D
    D <-->|fetch current date| I
    D -->|return date| C
    
    C -->|"detect: need day of week"| E
    E -->|calculate| C
    
    C -->|"detect: need location coords"| F
    F <-->|geocode location| J
    F -->|return coordinates| C
    
    C -->|"detect: need weather data"| G
    G <-->|fetch forecast| K
    G -->|return weather data| C
    
    C --> H
    H --> A
    
    classDef userClass fill:#00FFFF,stroke:#FFFFFF,stroke-width:2px,color:black
    classDef skClass fill:#00FF00,stroke:#FFFFFF,stroke-width:2px,color:black
    classDef pluginClass fill:#FFFF00,stroke:#FFFFFF,stroke-width:2px,color:black
    classDef externalClass fill:#FF9966,stroke:#FFFFFF,stroke-width:2px,color:black
    
    class A userClass
    class B,C,H skClass
    class D,E,F,G pluginClass
    class I,J,K externalClass
```

This diagram demonstrates how Semantic Kernel handles a complex user request by:

1. **Analyzing the user's query** to determine required information
2. **Automatically detecting** which plugin functions to call
3. **Orchestrating calls** to different plugins in the appropriate sequence
4. **Retrieving external data** through plugin functions that connect to real-world APIs
5. **Combining all information** to generate a comprehensive response

The plugin architecture allows the AI model to access real-time, specialized information that it couldn't otherwise obtain, significantly extending its capabilities beyond what's contained in its training data.

## Success Criteria

1. To complete this challenge successfully, the AI should be able to answer the following questions correctly:
   * What time is it?
   * What was the date 4 days ago?
   * What is the day of the week for the last day of next month?
   * What day of the week does today's date next year fall on?
2. In addition, you should have a second plugin that can can answer questions around the weather. You should now be able to get the Chatbot to answer the following questions:
    * What is today's weather in San Francisco?
    * Is it going to rain next week?
    * What's the high temp going to be today?
    * Do I need a raincoat for my movie date Friday?

## Learning Resources

*Semantic Kernel Resource Links:*

* [Plugins for Semantic Kernel](https://learn.microsoft.com/en-us/semantic-kernel/concepts/plugins/?pivots=programming-language-python)
* [Adding Native Plugins to Semantic Kernel](https://learn.microsoft.com/en-us/semantic-kernel/concepts/plugins/adding-native-plugins?pivots=programming-language-python)

* [Planning in Semantic Kernel](https://learn.microsoft.com/en-us/semantic-kernel/concepts/planning?pivots=programming-language-python)

* [Planning with Automatic Function Calling](https://devblogs.microsoft.com/semantic-kernel/planning-with-semantic-kernel-using-automatic-function-calling/)

*API Client Resources:*

* [Microsoft Recommendations for HTTP Client](https://learn.microsoft.com/en-us/dotnet/fundamentals/networking/http/httpclient-guidelines#recommended-use)


*API Docs:*

* [Geocoding API](https://geocode.maps.co/)

* [Open-Meteo API](https://open-meteo.com/en/docs)

## Tips

* Use a API test client such as Bruno or Postman to test the APIs  first, before writing code to connect it to your plugin.

## Advanced Challenges (Optional)

Too comfortable?  Eager to do more?  Try these additional challenges!

* Add debug logging messages to your Time plugin.

* Create a Plugin to pull data from a database instead of an API. You can find publicly available data sources here: [Database Star - Free Datasets](https://www.databasestar.com/free-data-sets/)

* Use the resources below to find a public API to build another plugin with.
  * [Public API Search Engine](https://publicapis.io/)
  * [Public API List on GitHub](https://github.com/public-api-lists/public-api-lists)
  * [Open API List](https://apilist.fun/)

* Look for APIs that are either current event related - such as  events and calendars - or something with esoteric knowledge that the AI does poorly with, such as the UPC database, or something like the [PokeAPI - Pokemon Pokedex API](https://pokeapi.co/docs/v2).

### [< Previous Challenge](./Challenge-02.md) - **[Home](../README.md)** - [Next Challenge >](./Challenge-04.md)
