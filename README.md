# TableLLM: Enabling Tabular Data Manipulation by LLMs in Real Office Usage Scenarios


We present [**T**able**LLM**](https://modelscope.cn/models/TABLELLM/TableLLM), a powerful large language model designed to handle tabular data manipulation tasks efficiently, whether they are embedded in spreadsheets or documents, meeting the demands of real office scenarios. The TableLLM are fine-tuned based on Llama3.1-8B.

TableLLM generates either a code solution or a direct text answer to handle tabular data manipulation tasks based on different scenarios. Code generation is used for handling spreadsheet-embedded tabular data, which often involves the insert, delete, update, query, merge, and chart operations of tables. Text generation is used for handling document-embedded tabular data, which often involves the query operation of short tables.

## Benchmark Details
We use six public benchmarks and one self-created benchmark for evaluation. As the public benchmarks we used are modified to fit the application scenario of TableLLM, we provide a detailed description of these public benchmarks and self-created benchmarks below. You can obtain the original file of these benchmarks in ```benchmark``` folder.
- WikiTQ: Limit the table to a token count of less than 500 and randomly sample 633 instances.
- TAT-QA: Limit the table to a token count of less than 500 and randomly sample 800 instances.
- FeTaQA: Limit the table to a token count of less than 500 and randomly sample 753 instances.
- OTTQA: Limit the table to a token count of less than 500 and use all instances that meet this condition.
- WikiSQL: As the WikiSQL testset contains incorrect answers and ambiguous questions, we manually filter out 1000 records and construct a subset of the WikiSQL testset called wikisql-human-annotated.
- Spider: As TableLLM currently focuses on single-table queries, we filter out single-table questions in Spider dev ser and also remove questions whose answers are empty.
- Self-created: We create a new benchmark, including the insert, delete, update, query, merge, and chart operations of tables. For more details, please refer to the paper.

## Prompt Template
The prompts we used for generating code solutions and text answers are introduced below.

### Code Solution
The prompt template for the insert, delete, update, query, and chart operations on a single table.
```
[INST]Below are the first few lines of a CSV file. You need to write a Python program to solve the provided question.

Header and first few lines of CSV file:
{csv_data}

Question: {question}[/INST]
```

The prompt template for the merge operation on two tables.
```
[INST]Below are the first few lines two CSV file. You need to write a Python program to solve the provided question.

Header and first few lines of CSV file 1:
{csv_data1}

Header and first few lines of CSV file 2:
{csv_data2}

Question: {question}[/INST]
```

The csv_data field is filled with the first few lines of your provided table file. Below is an example:
```
Sex,Length,Diameter,Height,Whole weight,Shucked weight,Viscera weight,Shell weight,Rings
M,0.455,0.365,0.095,0.514,0.2245,0.101,0.15,15
M,0.35,0.265,0.09,0.2255,0.0995,0.0485,0.07,7
F,0.53,0.42,0.135,0.677,0.2565,0.1415,0.21,9
M,0.44,0.365,0.125,0.516,0.2155,0.114,0.155,10
I,0.33,0.255,0.08,0.205,0.0895,0.0395,0.055,7
```

### Text Answer
The prompt template for direct text answer generation on short tables.
````
[INST]Offer a thorough and accurate solution that directly addresses the Question outlined in the [Question].
### [Table Text]
{table_descriptions}

### [Table]
```
{table_in_csv}
```

### [Question]
{question}

### [Solution][INST/]
````

## Environment Setup

Install the requirements with pip:
```
pip install -r requirements.txt
```

## Inference
The inference results of TableLLM are provided in ```inference/results``` folder. You can also obtain the inference result by yourself. The example commands of spreadsheet-embedded tabular data (e.g., WikiSQL) and document-embedded tabular data (e.g., WTQ) are shown below:
```
cd inference

python inference_code.py --dataset wikisql --model_path TableLLM-8b

python inference_text.py --dataset wtq --model_path TableLLM-8b
```

## Evaluation
The python code in ```evaluation``` folder is used for reproducing evaluation results. For code generation benchmarks, you can run the following command to reproduce the result of TableLLM-8b on WikiSQL:
```
cd evaluation/wikisql-eval
tar -zxvf csv_tables.tar.gz 
python eval.py --infer_data ../../inference/results/TableLLM-8b/Infer_wikisql.jsonl
```

For text generation, we use [CritiqueLLM](https://github.com/thu-coai/CritiqueLLM) for judgement. We also provide the judgement results running by ourselves. You can obtain it in ```inference/results``` folder and reproduce the results using the following command:
```
cd evaluation/text-eval
python get_sum_grade.py --grade_data ../../inference/results/TableLLM-8b/Grade_wtq.jsonl
```

## Deployment
You can use the code in ```deployment``` folder as the frontend and backend for deploying TableLLM.

![platform](images/platform.png)

Deploy TableLLM using vllm. Remember to modify the PORT and MODEL_PATH in the script and ```config.json```.
```
cd deployment
bash scripts/deploy_tablellm.sh
```

Install mongodb and change the username and password to yours in ```config.json```. Prepare the default tables and questions:
```
bash prepare_default.sh
```

Deploy the streamlit app:
```
streamlit run streamlit.py --server.port PORT
```
