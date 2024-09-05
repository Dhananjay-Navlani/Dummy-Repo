# Dummy-Repo

All main branch rules here
# Purpose
The purpose of BookMatch is to offer users an intuitive and dynamic way to discover new books tailored to their tastes. By combining user input with the power of generative AI, BookMatch aims to provide highly relevant and enjoyable book suggestions.

# Workflow
1. Genre Selection:
- Users start by opting out of genres they are not interested in.
- Initial recommendations are generated based on the selected genres.

2. Recommendations:
- Users receives a list of book recommendations.
- Users can like, dislike, and rate books.
- Users can change the genres selected anytime.
- Based on user feedback (likes, dislikes, ratings), new recommendations are generated using the LLM.

3. Filtering Recommendations:
- Users can filter and access old recommendations.
- Recommendations are stored and can be revisited at any time.

# Tech Stack 
- **Kotlin Multiplatform (KMP)**:
   - Shared business logic is written in Kotlin.
   - Native UI for Android using Jetpack Compose.
   - Native UI for iOS using SwiftUI.

- **Backend**:
   - Supabase is used for backend services.

- **Generative AI**:
   - OpenAI is used for generating book information.

- **Authentication**:
   - Google OAuth is used for authentication in both the Android and iOS apps via Supabase.
 
# Project Structure
### [Shared Module](../main/shared/src/commonMain/kotlin) 
<img src="https://github.com/user-attachments/assets/ef4e9431-87de-4ed2-b4ce-16b96a02edaf" width=500/>

- Contains business and shared logic for both apps i.e. composeApp and iosApp

|Directory | Description | Important file
| --- | --- | ---
|**api** | Contains API client files for accessing external service like OpenAi, Gemini | *‘GeminiClient.kt’*, *‘OpenAiClient.kt’*
**data** | Contains file for database operations and authentication operations | _‘RemoteDataSource.kt’_, _‘SupabaseProvider.kt’_
**model** | Defines model file to parse the request and response from AI as well as database tables | _‘OpenAiRequest.kt’, ‘OpenAiResponse.kt’, ‘SupabaseRemoteEntities.kt’_
**utils** | Contains Constants classes which store api url and keys and extension functions | _‘Constants.kt’_

- **Note:**

  - The [Constants.kt](../main/shared/src/commonMain/kotlin/utils/Constants.kt) file includes the following properties which you can update with their own values: 
      - ```OAUTH_WEB_CLIENT_ID```: Google web client client id generated from google cloud console to use sign in with           google feature
      - ```OPEN_AI_API_KEY```: Your OpenAI api key
      - ```GEMINI_API_KEY```: Your Gemini api key (if you want to use gemini instead of OpenAi along with GeminiClient)
  - The [GeminiClient.kt](../main/shared/src/commonMain/kotlin/api/GeminiClient.kt) file is included for those who want to use Gemini for book recommendation generation. Replace ```GeminiClient```'s functions with ```OpenAiClient``` in ```MainViewModel.kt``` of composeApp and SharedViewModel.swift of iosApp
  - You can edit ```systemInstruction``` property inside [OpenAiClient](../main/shared/src/commonMain/kotlin/api/OpenAIClient.kt)/[GeminClient](../main/shared/src/commonMain/kotlin/api/GeminiClient.kt)  to customize the AI response according to your own needs.

 ### [Android UI](../main/composeApp/src/androidMain/kotlin/com/novumlogic/bookmatch)
<img src="https://github.com/user-attachments/assets/33e282b3-4f06-4cf3-96f5-ecfebce61b2d"/>

- The Android UI specific code is written under composeApp module's androidMain with jetpack compose. It contains code for all screens along with viewmodel which connect the data layer i.e. shared module to UI layer

### [ios UI](../main/iosApp/iosApp)
<img src="https://github.com/user-attachments/assets/950e3231-b41a-4c49-af58-ea99882dd8d0" /> 

- The ios UI specific code is written under iosApp with swiftui along with viewmodel which connects data layer i.e. shared module to UI layer


# Backend Structure
### Database tables 

1. **Users**
- Used to store user’s info and also maintains certain info from their login session.

| Fields                    | Datatype   | Constraint                                      | Description                                                     |
|---------------------------|------------|-------------------------------------------------|-----------------------------------------------------------------|
| user_id                   | UUID       | primary-key, default = auth.uid()               | unique identifier for user                                      |
| email                     | text       | not null, unique                                | user's email provided via Google OAuth                         |
| display_name              | text       | not null                                        | username provided via Google OAuth                             |
| avatar_url                | text       |                                                 | user's profile photo provided via Google OAuth                 |
| category_shown            | boolean    |                                                 | Describes if the user was shown category selection screen in the login session |
| selected_categories       | text       |                                                 | Describes the selected categories/genres by the user           |
| last_recommendation_time  | int8       |                                                 | Describes the last recommendation generation timestamp of user |
| created_at                | timestampz | not null                                        | Stores the first time the account was logged in the app        |



2. **Categories** (Genres)
- To store the genres/categories of books from which user can generate his recommendation

| Fields          | Datatype | Constraint  | Description                            |
|-----------------|----------|-------------|----------------------------------------|
| category_id     | int2     | primary-key | uniquely identify each genre/category  |
| category_name   | text     | not-null    | name of each genre                     |
| category_emoji  | text     |             | emoji for each genre                   |

3. **Books**
- To store generated book’s information

| Fields                  | Datatype | Constraint                                                                                      | Description                                                                 |
|-------------------------|----------|-------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------|
| book_id                 | int8     | primary-key                                                                                    | unique identifier                                                           |
| book_name               | text     | not null, unique                                                                               | book title                                                                  |
| author_name             | text     | not null, unique                                                                               | author name                                                                 |
| genre_tags              | text[]   |                                                                                                 | applicable genre tags                                                       |
| category_id             | int2     | not null, foreign key referring category_id of categories table with on update cascade, on delete cascade | refers to the category in the categories table                              |
| description             | text     | not null                                                                                        | 2 line description                                                          |
| pages                   | text     | not null                                                                                        | number of pages in the book                                                 |
| isbn                    | text     | not null                                                                                        | unique ISBN number to fetch book cover using Open Library Book Cover API    |
| first_date_of_publication | text   | not null                                                                                        | first date of publication                                                   |
| reference_link          | text     | not null                                                                                        | URL to open book-related webpage                                            |


4. **Recommendations**
- To store the recommendation generated by each user

| Fields           | Datatype | Constraint                                                                                               | Description                                                     |
|------------------|----------|----------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------|
| recommendation_id | int8     | primary-key                                                                                              | unique identifier for recommendations                           |
| timestamp        | int8     | not null                                                                                                 | stores UNIX epoch time                                          |
| user_id          | UUID     | not null, foreign key referring to users table user_id field with on update cascade, on delete cascade, default = auth.uid() | uniquely identifies which recommendation belongs to which user  |

5. **Recommended_books**
- To store the books generated in each recommendation seperately


| Fields               | Datatype | Constraint                                                                                      | Description                                                     |
|----------------------|----------|-------------------------------------------------------------------------------------------------|-----------------------------------------------------------------|
| id                   | int8     | primary-key                                                                                    | uniquely identifies each book for a given recommendation_id     |
| recommendation_id    | int8     | not null, foreign key referring to users table user_id with on update cascade, on delete cascade | to access corresponding info in the recommendation table        |
| book_id              | int8     | foreign key referring to books table book_id field with on update cascade, on delete cascade     | to access the book’s info from the books table                  |
| liked                | boolean  |                                                                                                 | indicates if the book is liked by the user                      |
| rating               | int2     | check value is between 0 and 6                                                                  | rating between 1 to 5 given by the user                         |
| last_updated_time    | int8     | default = 0                                                                                     | last time the user performed any interaction                    |
| read                 | boolean  | default = false                                                                                 | indicates if the user has read the book                         |

6. **Chat_history**
- To store the chat_history which is used to provide context when sending chat_completion request to LLM
  
| Fields      | Datatype | Constraint                                             | Description                                                     |
|-------------|----------|--------------------------------------------------------|-----------------------------------------------------------------|
| id          | int8     | primary-key                                            | unique identifier                                               |
| timestamp   | int8     | not null                                               | stores UNIX epoch time                                          |
| user_id     | UUID     | foreign key referring to users table user_id field     | identifies to which user the chat history belongs               |
| user_text   | text     |                                                        | user message sent to OpenAI or any other LLM’s API request      |
| ai_answer   | text     |                                                        | response received from AI                                       |


- Overview
  - A user can have many recommendations (one to many) 
  - A recommendation can have many recommended_books (one to many)
  - A recommended_book has one to one relation with book (one to one) 
  - A user can have many chat_history data (one to many) 

# Getting Started

## Prerequisites
- [Kotlin Multiplatform Mobile](https://www.jetbrains.com/help/fleet/getting-started-with-kotlin-multiplatform.html)
- [Jetpack Compose](https://developer.android.com/compose)
- [SwiftUI](https://developer.apple.com/xcode/swiftui/)
- [Supabase](https://supabase.com)
- [OpenAI chat completions API](https://platform.openai.com/docs/guides/chat-completions)
