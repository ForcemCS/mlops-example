# House Price Predictor – An MLOps Learning Project

这是一个非常标准的 端到端机器学习项目结构，涵盖了从原始数据处理到模型部署的全流程

## Project Structure

```
house-price-predictor/
├── data
│   ├── processed
│   │   └── README.md
│   └── raw
│       └── house_data.csv   ##数据工程师完成大量的预处理后得到的
├── models
│   └── trained
│       └── README.md
├── notebooks
│   ├── 00_data_engineering.ipynb
│   ├── 01_exploratory_data_analysis.ipynb
│   ├── 02_feature_engineering.ipynb
│   └── 03_experimentation.ipynb
├── src
│   ├── api
│   │   ├── README.md
│   │   ├── inference.py
│   │   ├── main.py
│   │   ├── requirements.txt
│   │   ├── schemas.py
│   │   └── utils.py
│   ├── data
│   │   └── run_processing.py     
│   ├── features
│   │   └── engineer.py
│   └── models
│       └── train_model.py
└── streamlit_app
    ├── README.md
    ├── app.py
    └── requirements.txt

```
目录结构深度解析
   * `data/` (数据层)
       * raw/: 存放原始数据（如 house_data.csv）。在 MLOps 中，原始数据应视为只读，不可修改。
       * processed/: 存放清洗、转换后的数据，供模型训练直接读取。通过notebook下的代码清洗
   * `notebooks/` (实验与探索层)
       * 按序号排列（00-03），代表了数据科学家的工作流：数据工程 -> 探索性数据分析 (EDA) -> 特征工程 -> 模型实验。这是项目的“实验室”。
   * `src/` (核心源代码层)
       * 这是项目的核心，将 Notebook 中的实验代码工程化、模块化。
       * data/run_processing.py: 数据清洗脚本。
       * features/engineer.py: 特征提取逻辑（确保训练和推理时特征一致）。
       * models/train_model.py: 自动化训练脚本，输出模型文件。
       * api/: 模型推理服务。使用 FastAPI 或 Flask 将模型包装成 Web API，供其他系统调用。
   * `models/trained/` (模型仓库)
       * 存放训练好的模型权重和配置（如 .pkl, .h5, .onnx）。
       * 模型配置是数据科学家与机器学习工程师协作的关键
   * `deployment/` (部署与运维层)
       * mlflow/: 用于实验跟踪和模型版本管理。
       * kubernetes/: 生产环境容器化部署的配置文件。
   * `streamlit_app/` (应用演示层)
       * 一个基于 Python 的前端展示界面，让非技术用户也能通过网页输入参数并查看预测结果。

---

## MLOps 各阶段的协作流程

  这个结构体现了如何将一个“算法模型”转化为“软件产品”：


  阶段 A：数据准备 (Data Engineering)
   * 执行： 运行 src/data/run_processing.py。
   * 协作： 从 data/raw 读取数据，经过 src/features/engineer.py 处理，将结果存入 data/processed。


  阶段 B：实验与开发 (Experimentation)
   * 执行： 数据科学家在 notebooks/ 中快速迭代。
   * 协作： 确定最佳特征和模型算法后，将逻辑同步到 src/ 下的 Python 模块中，实现从脚本到代码的转化。


  阶段 C：模型训练与版本化 (CI/CD for ML)
   * 执行： 运行 src/models/train_model.py。
   * 协作： 训练脚本会调用 src/features 中的特征逻辑。如果配置了 deployment/mlflow，训练过程中的参数、指标和最终模型会被自动记录到 MLflow 服务器中。


  阶段 D：部署与推理 (Deployment & Inference)
   * 执行： 启动 src/api/main.py 或部署 deployment/ 中的配置。
   * 协作： API 服务从 models/trained/ 加载模型。当用户发送请求时，API 调用 src/api/utils.py 进行预处理，并返回预测价格。


  阶段 E：交互演示 (UI/Presentation)
   * 执行： 运行 streamlit_app/app.py。
   * 协作： Streamlit 作为客户端，向 src/api/ 发送 HTTP 请求，获取预测结果并可视化展示给终端用户。

## Model Workflow

### Step 1: Data Processing

清理和预处理原始房屋数据集

```
python src/data/run_processing.py   --input data/raw/house_data.csv   --output data/processed/cleaned_house_data.csv
```

------

### Step 2: Feature Engineering

应用转换并生成特性

```
python src/features/engineer.py   --input data/processed/cleaned_house_data.csv   --output data/processed/featured_house_data.csv   --preprocessor models/trained/preprocessor.pkl
```

------

### Step 3: Modeling & Experimentation

训练你的模型并将所有内容记录到MLflow中：

```
python src/models/train_model.py   --config configs/model_config.yaml   --data data/processed/featured_house_data.csv   --models-dir models   --mlflow-tracking-uri http://12.0.0.200:5555
```
