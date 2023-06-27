# Q&A for PDF Documents

This is a simple Streamlit application that makes use of LangChain and OpenAI's LLM to provide a Q&A for PDF documents.

Users can upload a PDF document, and the LLM will answer questions about the document's content.

## Usage

To use this application:

1. Create a virtual environment and install the requirements.

    ```bash
    pip install -r requirements.txt
    ```

2. Obtain an [OpenAI API key](https://platform.openai.com/account/api-keys).

    After obtaining your OpenAI API key, create a `.env` file in the root of your project directory.

3. Add the following line to the `.env` file:

   ```text
   OPENAI_API_KEY=your_openai_api_key
   ```

   Replace `your_openai_api_key` with your actual OpenAI API key. The application is set to automatically load environment variables from the `.env` file.

4. Start a [redis-stack](https://redis.io/docs/stack/) docker container as a [vector database](https://redis.com/solutions/use-cases/vector-database/).

    ```bash
    docker run -d --name redis-stack-server -p 6379:6379 redis/redis-stack-server:latest
    ```

5. Run the application:

   ```bash
   streamlit run app.py
   ```

6. Open the Streamlit interface in your web browser.

    ```text
    http://localhost:8501
    ```

7. Upload a PDF file and ask questions about its content. The chatbot will generate answers based on the document's content.

    - [Sample PDF file about GPT4All.](https://s3.amazonaws.com/static.nomic.ai/gpt4all/2023_GPT4All_Technical_Report.pdf)

    - Sample questions "Who are the authors of this paper?", "What was the cost for training GPT4All?", "Which model is GPT4All based on?".

## Architecture

The app works in several steps:

1. **Upload PDF**: The user uploads the PDF file to ask questions about.

2. **Text Extraction**: The app uses the PyPDF2 library to read the PDF file and extract text from it.

3. **Text Splitting**: It splits the text into smaller chunks to overcome token limit issue and understand the content.

4. **Embeddings Creation**: Using OpenAIEmbeddings, the app creates text embeddings from the chunks.

5. **Vector database creation**: After running the redis-stack docker container, the app uses it as a [vector database](https://python.langchain.com/docs/modules/data_connection/vectorstores/integrations/redis) to store the embeddings obtained from the previous step.

6. **Vector similarity search**: When a query about the document is entered, the app will create an embedding of the query and use it to perform a vector similarity search to find the data points that are similar to the query vector in the redis vector database.

7. **Chain creation**: A Q&A chain is created using the OpenAI model and the results from the similarity search and the query are passed to the OpenAI model for procressing via its API.

8. **Results**: Finally the app will provide a response based on the contents of the uploaded PDF and the query.
