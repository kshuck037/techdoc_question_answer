# Architecture Overview
![Architecture](diagram/TechDocQuestionAnswer.drawio.png)

The architecture follows a simple yet modular design approach. This approach makes the system easy to understand and modify as needs change. The data pipeline has two parts: file parsing and data chunking, and a vector DB with embeddings. The system has an LLM interface that pulls context from the vector database based on the question and sends the question along with the context. The LLM forms an answer and provides a JSON document with the question, answer, and source documents.
# Implementation Details

The file parsing and chunking are accomplished with the unstructured Python library. I chose this library because it handled two pipeline steps in a single call and supported several file formats, including PDF, TXT, and HTML. The vector database is ChromaDB; when a document is added, it generates embeddings using the all-MiniLM-L6-v2 model via ONNX/Sentence-Transformers. You can upgrade to an OpenAI embedding model to improve quality. However, I didn't do this because it wasn't local or free. The semantic search runs within the vector database using Euclidean distance. ChromaDB was selected because it is a simple, embedded setup that uses local resources. The LLM is a local Ollama instance; it costs nothing and can be swapped for the OpenAI LLM API. The LLM model is Llama 3.1 8 B. 

# Code Usage
You can run the included notebook in Google Colab using the Run All command. You may need to run the pip install once before running the entire notebook. Google Drive is used for the file location. However, the code details are below, so you can use it in other environments.\  
- File Parsing and Data Chunking
The following function handles file parsing and data chunking. It takes a file and outputs a chunked document.
```python
process_file(filename)
```
- Vector DB with embeddings
The vector database is in the VectorDatabase class and has two methods, add_documents which takes a chunked document and puts it into the database and query_documents takes a str question and returns a set of documents related to the question.
- Q&A LLM
The QuestionsSystem class has one method, question_answering, that takes a question and returns a response along with the related documents. 
- System integration
The following function processes a set of questions and returns a JSON-formatted string
```python
process_questions(questions, folder, vector_db)
```

## Considerations
The current code parameters are hardcoded and can be moved into a YAML file for easier maintenance and editing. The data pipeline runs on each execution, so the vector DB is rebuilt each time; this is done in Google Colab because instances delete data on disconnect. However, it should be set up to do so via a switch variable or auto-detected. 
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
    "answer": "The cost of LEO (Low Earth Orbit) is not explicitly stated in the context. However, it is mentioned that a book titled \"LEO on the Cheap: Methods for Achieving Drastic Reductions in Space Launch Costs\" was written by London, John R. in 1994. This suggests that the book explores ways to reduce costs associated with LEO, but the actual cost of LEO is not provided.",
    "source": "Cost-Reduction-paper.pdf Cost\nCost-Reduction-paper.pdf REFERENCES\nCost-Reduction-paper.pdf Orbit Cost\nCost-Reduction-paper.pdf Very low cost, continuous communications link"
  },
  {
    "question": "What is a small satellite?",
    "answer": "According to the context, a small satellite is referred to as a \"SmallSat\". A SmallSat is a small satellite that gives up global coverage for much better coverage of a specific latitude range.",
    "source": "Cost-Reduction-paper.pdf REFERENCES\nLEO_LaunchCosts_Aug2022.pdf All satellites\nCost-Reduction-paper.pdf M3. Use Repeat Coverage Orbit\nCost-Reduction-paper.pdf REFERENCES"
  },
  {
    "question": "Why use small satellites?",
    "answer": "Small satellites can be used because they are lower cost, can reduce mission risk, and allow for better coverage of a specific latitude range.",
    "source": "Cost-Reduction-paper.pdf REFERENCES\nCost-Reduction-paper.pdf Pg1. Operational SmallSats\nCost-Reduction-paper.pdf M3. Use Repeat Coverage Orbit\nLEO_LaunchCosts_Aug2022.pdf All satellites"
  },
  {
    "question": "What steps have been taken to reduce launch costs?",
    "answer": "Unfortunately, the provided context doesn't specify the steps taken to reduce launch costs. However, it does mention a book titled \"Reducing Space Mission Cost\" by James R. Wertz and Wiley J. Larson, which includes a chapter \"Reducing Launch Cost\" by John R. London.",
    "source": "Cost-Reduction-paper.pdf Missions\nCost-Reduction-paper.pdf Reducing Space Mission Cost\u2014Launch\nCost-Reduction-paper.pdf REFERENCES\nCost-Reduction-paper.pdf 2.7 Launch"
  },
  {
    "question": "What is the weight of a small satellite?",
    "answer": "Unfortunately, the context does not provide a specific weight for a small satellite. It mentions \"cost per kg\" in Table A5, but it does not give a specific weight for a small satellite.",
    "source": "Cost-Reduction-paper.pdf REFERENCES\nCost-Reduction-paper.pdf REFERENCES\nLEO_LaunchCosts_Aug2022.pdf All satellites\nLEO_LaunchCosts_Aug2022.pdf Std.Dev"
  }
]
```

# Acknowledgment of AI Usage

AI was used for documentation formatting and grammar checking via Grammarly. Google AI was used to research which libraries are available to use for each component of the system. It was also used to understand library code usage, provide example code, and provide a basic framework. Google AI helped troubleshoot code implementation issues, including error messages and proposed code fixes. 
