# Architecture Overview
![Architecture](diagram/TechDocQuestionAnswer.drawio.png)

The architecture follows a simple, modular design. This approach makes the system easy to understand and modify as needs change. The data pipeline has two parts: file parsing and data chunking, and a vector DB with embeddings. The system has an LLM interface that pulls context from the vector database based on the question and sends the question and context to the LLM. The LLM forms an answer and provides a JSON document with the question, answer, and source documents.
# Implementation Details

The file parsing and chunking are accomplished with the unstructured Python library. I chose this library because it handled two pipeline steps in a single call and supported several file formats, including PDF, TXT, and HTML. The vector database is ChromaDB; when a document is added, it generates embeddings using the all-MiniLM-L6-v2 model via ONNX/Sentence-Transformers. You can upgrade to an OpenAI embedding model to improve quality. However, I didn't do this because it wasn't local or free. The semantic search runs within the vector database using Euclidean distance. I selected ChromaDB because it is a simple, embedded setup that uses local resources. The LLM is a local Ollama instance; it costs nothing and can be swapped for the OpenAI LLM API. The LLM model is Llama 3.1 8 B. 

# Code Usage
You can run the included notebook in Google Colab using the Run All command. You may need to run the pip install once before running the entire notebook. Google Drive is used for the file location. However, the code details are below, so you can use it in other environments.\  
- File Parsing and Data Chunking
The following function handles file parsing and data chunking. It takes a file and outputs a chunked document.
```python
process_file(filename)
```
- Vector DB with embeddings
The VectorDatabase class has two methods: add_documents, which takes a chunked document and adds it to the database, and query_documents, which takes a str question and returns a set of documents related to the question.
- Q&A LLM
The QuestionsSystem class has one method, question_answering, that takes a question and returns a response along with the related documents. 
- System integration
The following function processes a set of questions and returns a JSON-formatted string
```python
process_questions(questions, folder, vector_db)
```

## Considerations
The current code hardcodes parameters, which can be moved into a YAML file for easier maintenance and editing. The data pipeline runs on each execution, so the vector DB is rebuilt each time; this is done in Google Colab because instances delete data on disconnect. However, it should be set up to do so via a switch variable or auto-detected. 
# Example Usage
Developed and ran the notebook in Google Colab. The Google Colab compute instance was an A100. The instance type supports the local LLM used.

## Reference Data
The papers are located in the ./files directory of this repo. 
- Crisp, Nicholas & Smith, Katharine & Hollingsworth, Peter. (2014). Small Satellite Launch to LEO: A Review of Current and Future Launch Systems. TRANSACTIONS OF THE JAPAN SOCIETY FOR AERONAUTICAL AND SPACE SCIENCES, AEROSPACE TECHNOLOGY JAPAN. 12. Tf_39-Tf_47. 10.2322/tastj.12.Tf_39. 
- González-González, Rubén, García Pérez, Andrés Y Alonso, G. (2022). Small Satellites Launcher Design Efficiency Improvement through the Application of Multidisciplinary Modeling and Simulation-Based Systems Engineering.
- Adilov, Nodir & Albertson, Nikolas & Alexander, Peter & Cunningham, Brendan. (2021). An Economic Analysis of Launch Cost Reductions for Low Earth Orbit Satellites.
- Maklisuwan, Settapong & Kanchanarat, Borwonrat. (2022). Small Satellites for Low-Cost Space Access: Launch, Deployment, Integration, and In-Space Logistics. American Journal of Industrial and Business Management. 12. 1480-1497. 10.4236/ajibm.2022.1210082. 
- Savvina, Elena & Panasenkov, Valery. (2017). Affordable Small Satellites Launch Options Research.
- Wertz, J., Conger, R., Rufer, M., Sarzi-Amadé, N., & Van Allen, R. (2011). Methods for achieving dramatic reductions in space mission cost. https://s3vi.ndc.nasa.gov/ssri-kb/static/resources/Cost-Reduction-paper.pdf

## Example Questions
- What is the cost of LEO?
- What is a small satellite?
- Why use small satellites?
- What steps have been taken to reduce launch costs?
- What is the weight of a small satellite?
## Example Output
```json
[
  {
    "question": "What is the cost of LEO?",
    "answer": "The cost of LEO (Low Earth Orbit) is not explicitly stated in the provided context, but it is mentioned that drastic reductions in space launch costs can be achieved using certain methods, as described in the book \"LEO on the Cheap\" by London, John R.",
    "source": "Cost-Reduction-paper.pdf Cost\nCost-Reduction-paper.pdf REFERENCES\nCost-Reduction-paper.pdf Orbit Cost\nCost-Reduction-paper.pdf Very low cost, continuous communications link"
  },
  {
    "question": "What is a small satellite?",
    "answer": "The context doesn't directly define what a small satellite is. However, based on the context, it appears that a small satellite is a satellite that is part of a small satellite program or mission, which is implied to be a cost-effective and possibly more efficient alternative to traditional satellite missions. The reference to \"Small Satellites: Past, Present, and Future\" edited by Helvajian and Janson, and the mention of \"SmallSat Utility\" and \"Small Satellite Launch\" in the context suggest that small satellites refer to a category of satellites that are smaller in size or complexity compared to traditional satellites.",
    "source": "Cost-Reduction-paper.pdf REFERENCES\nLEO_LaunchCosts_Aug2022.pdf All satellites\nCost-Reduction-paper.pdf REFERENCES\nCost-Reduction-paper.pdf flexibility"
  },
  {
    "question": "Why use small satellites?",
    "answer": "Small satellites can be more cost-effective and reduce mission risk by allowing satellite replacement and multiple satellites on orbit. They can also serve as a testing ground for technology that will be used in larger missions.",
    "source": "Cost-Reduction-paper.pdf Pg1. Operational SmallSats\nCost-Reduction-paper.pdf REFERENCES\nLEO_LaunchCosts_Aug2022.pdf All satellites\nCost-Reduction-paper.pdf L5. Low-Cost Small Launch"
  },
  {
    "question": "What steps have been taken to reduce launch costs?",
    "answer": "Unfortunately, the context only provides a reference to a book chapter and a website, but it does not explicitly state the steps taken to reduce launch costs. However, it does mention that the topic is discussed in more detail in the book chapter by London (1996) and on the Reinventing Space website (2011).",
    "source": "Cost-Reduction-paper.pdf Missions\nCost-Reduction-paper.pdf Reducing Space Mission Cost\u2014Launch\nCost-Reduction-paper.pdf REFERENCES\nCost-Reduction-paper.pdf 2.7 Launch"
  },
  {
    "question": "What is the weight of a small satellite?",
    "answer": "The text does not mention the weight of a small satellite. It discusses references related to small satellites, including conferences, publications, and courses, but it does not provide information about their weight.",
    "source": "Cost-Reduction-paper.pdf REFERENCES\nCost-Reduction-paper.pdf REFERENCES\nCost-Reduction-paper.pdf flexibility\nLEO_LaunchCosts_Aug2022.pdf All satellites"
  }
]
```

# Acknowledgment of AI Usage

AI was used for documentation formatting and grammar checking via Grammarly. Google AI was used to research which libraries are available for each system component. It was also used to understand library code usage, provide example code, and provide a basic framework. Google AI helped troubleshoot implementation issues, including error messages, and suggested code fixes. 
