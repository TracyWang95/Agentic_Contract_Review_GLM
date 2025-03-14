# Agentic_Contract_Review_GLM 项目

## 项目简介

Agentic_Contract_Review_GLM 是一个基于人工智能的合同审查工作流项目，旨在自动化审查合同是否符合特定的法规要求（以GDPR为例）。该项目通过解析合同中的关键条款，并与法规库中的相关条款进行匹配，生成合规性报告。本项目基于LlamaIndex项目二开而来，同时避免使用了LlamaParse这一SaaS组件。项目使用了LlamaIndex、VectorStoreIndex和OpenParses、智源bge-small-en-v1.5嵌入模型以及智谱AI GLM模型，其中VectorStoreIndex、OpenPares、bge-small-en-v1.5均为开源组件，GLM模型为API调用（也可以切换成GLM-9B开源模型以实现全量本地化部署），实现智能审核功能。

## 主要业务功能

1. **合同解析**：从合同中提取结构化数据，识别关键条款。
2. **条款匹配**：将合同条款与 GDPR 法规库中的相关条款进行匹配。
3. **自动化工作流**：通过定义的工作流自动化整个合同审查过程。
4. **合规性检查**：评估每个条款是否符合 GDPR 要求，并生成合规性报告。

## 核心技术栈

- **LlamaIndex**：用于构建和管理文档索引，支持高效的文档检索。
- **OpenParse**：用于解析复杂的 PDF 文档，提取文本内容。
- **BGE**：用于做文本向量化。
- **GLM**：用于实现大模型智能审核。

## 项目结构

- **agentic_contract_workflow_glm.ipynb**：主脚本，包含合同审查工作流实现的Demo。
- **data/**：存储 GDPR 法规文档和其他相关数据。
- **logs.py**：日志管理模块，负责日志的配置和输出。
- **try_parse_json_object.py**：JSON 解析工具，用于处理和修复从 LLM 返回的 JSON 数据。

## 辅助功能组件

### 日志管理

项目通过 `logs.py` 模块实现了日志的配置和管理。日志文件会根据时间戳创建新的目录，并支持日志文件的轮转和备份。日志级别可以通过配置进行调整，支持 `DEBUG`、`INFO`、`WARNING`、`ERROR` 和 `CRITICAL` 等级别。

#### 日志配置

日志配置文件通过 `get_config_dict` 函数生成，支持以下配置：

- **日志级别**：可以通过 `log_level` 参数设置日志级别。
- **日志文件路径**：通过 `log_file_path` 参数指定日志文件的存储路径。
- **日志轮转**：支持日志文件的轮转，通过 `log_max_bytes` 和 `log_backup_count` 参数控制日志文件的大小和备份数量。

#### 日志输出

日志会同时输出到控制台和文件中，方便调试和问题排查。

### JSON 解析工具

项目通过 `try_parse_json_object.py` 模块提供了 JSON 数据的解析和修复功能。该模块能够处理从 LLM 返回的 JSON 数据，修复潜在的格式问题，并确保 JSON 数据能够被正确解析为 Python 字典。

#### 具体功能

- **JSON 修复**：通过 `repair_json` 函数修复格式错误的 JSON 字符串。
- **AST 解析**：通过 `try_parse_ast_to_json` 函数解析包含 JSON 数据的字符串，并提取出 JSON 对象。
- **JSON 清理**：清理 JSON 字符串中的多余字符，如多余的引号、换行符等。

## 安装与运行

### 安装依赖

在运行项目之前，请确保已安装所需的 Python 包。可以通过以下命令安装依赖：

```python
pip install openparse[ml]
pip install llama-index llama-index-indices-managed-llama-cloud llama-cloud llama-parse
pip install llama-index-llms-huggingface
pip install llama-index-embeddings-huggingface json_repair llama-index-utils-workflow
pip install llama_index.llms.zhipuai
```

### 运行项目

1. **下载 GDPR 文档**：

   项目需要 GDPR 文档作为法规库。可以通过以下命令下载：

   ```bash
   mkdir -p data
   wget "https://eur-lex.europa.eu/legal-content/EN/TXT/PDF/?uri=CELEX:32016R0679" -O data/gdpr.pdf
   ```

2. **设置索引和解析器**：

   项目使用 LlamaIndex 构建文档索引，并使用 OpenParse 解析合同文档。

3. **设置类**

   #### 类与功能详解

   #### a. **ContractClause 类**

   `ContractClause` 类用于表示合同中的单个条款。每个条款包含以下字段：

   - **clause_text**：条款的文本内容。
   - **mentions_data_processing**：是否涉及个人数据的收集或使用。
   - **mentions_data_transfer**：是否涉及个人数据的传输，尤其是向第三方或跨境传输。
   - **requires_consent**：是否明确要求用户同意数据处理活动。
   - **specifies_purpose**：是否明确指定数据处理或传输的目的。
   - **mentions_safeguards**：是否提及数据的安全措施或其他保护措施。
   
   #### b. **ContractExtraction 类**

   `ContractExtraction` 类用于表示从合同中提取的结构化数据。包含以下字段：

   - **vendor_name**：供应商的名称（如果可识别）。
   - **effective_date**：合同的生效日期（如果可用）。
   - **governing_law**：合同的管辖法律（如果声明）。
   - **clauses**：从合同中提取的条款列表。
   
   #### c. **GuidelineMatch 类**
   
   `GuidelineMatch` 类用于表示与合同条款匹配的 GDPR 指南。包含以下字段：
   
   - **guideline_text**：与合同条款最相关的 GDPR 指南文本。
   - **similarity_score**：指南与条款的相似度评分（0 到 1 之间）。
   - **relevance_explanation**：指南与条款相关性的简要解释。
   
   #### d. **ClauseComplianceCheck 类**
   
   `ClauseComplianceCheck` 类用于表示合同条款的合规性检查结果。包含以下字段：
   
   - **clause_text**：合同条款的文本内容。
   - **matched_guideline**：与条款匹配的 GDPR 指南。
   - **compliant**：条款是否符合 GDPR 要求。
   - **notes**：附加的注释或建议。
   
   #### e. **ComplianceReport 类**
   
   `ComplianceReport` 类用于表示最终的合规性报告。包含以下字段：
   
   - **vendor_name**：供应商的名称（如果可识别）。
   - **overall_compliant**：合同是否总体符合 GDPR 要求。
   - **summary_notes**：对合同合规性的总结和建议。
   
   #### f. **ContractReviewWorkflow 类**
   
   `ContractReviewWorkflow` 类是合同审查工作流的核心类，负责整个审查流程的执行。主要方法包括：
   
   - **parse_contract**：解析合同并提取结构化数据。
   - **dispatch_guideline_match**：为每个合同条款分派指南匹配任务。
   - **handle_guideline_match**：处理条款与指南的匹配，并生成合规性检查结果。
   - **gather_guideline_match**：收集所有条款的合规性检查结果。
   - **generate_output**：生成最终的合规性报告。

4. **可视化工作流**

项目还支持将工作流可视化，生成 HTML 文件以便更好地理解工作流的执行过程：

```python
from llama_index.utils.workflow import draw_all_possible_flows
draw_all_possible_flows(ContractReviewWorkflow, filename="contract_workflow.html")
```
   
5. **运行工作流**

   通过以下命令运行合同审查工作流：

   ```python
   workflow = ContractReviewWorkflow(
       parser=parser,
       guideline_retriever=retriever,
       llm=llm,
       verbose=True,
       timeout=None,  # 不设置超时以确保工作流完成
   )

   handler = workflow.run(contract_path="/content/vendor_agreement.md")
   async for event in handler.stream_events():
       if isinstance(event, LogEvent):
           if event.delta:
               print(event.msg, end="")
           else:
               print(event.msg)

   response_dict = await handler
   print(str(response_dict["report"]))
   ```
   
## 审核过程示例
```json
  {
  "clause_text": "Vendor shall implement appropriate measures including: Encryption of Personal Data in transit and at rest, Access controls and authentication, Regular security testing and assessments, Employee training on data protection, Incident response procedures",
  "matched_guideline": {
    "guideline_text": "standard data-protection clauses in a wider contract... Controllers and processors should be encouraged to provide additional safeguards via contractual commitments that supplement standard protection clauses.",
    "similarity_score": 0.7,
    "relevance_explanation": "The guideline suggests that additional safeguards should be provided in contracts, which aligns with the vendor's clause on implementing appropriate measures."
  },
  "compliant": true,
  "notes": "The clause aligns well with the guideline, ensuring that appropriate measures for data protection are in place."
 }
```

```python
合规性检查成功: clause_text='Vendor shall implement appropriate measures including: Encryption of Personal Data in transit and at rest, Access controls and authentication, Regular security testing and assessments, Employee training on data protection, Incident response procedures' matched_guideline=GuidelineMatch(guideline_text='standard data-protection clauses in a wider contract... Controllers and processors should be encouraged to provide additional safeguards via contractual commitments that supplement standard protection clauses.', similarity_score=0.7, relevance_explanation="The guideline suggests that additional safeguards should be provided in contracts, which aligns with the vendor's clause on implementing appropriate measures.") compliant=True notes='The clause aligns well with the guideline, ensuring that appropriate measures for data protection are in place.'
Step handle_guideline_match produced event MatchGuidelineResultEvent
```
  
## 输出示例

项目最终会生成一个合规性报告，包含以下内容：

- **供应商名称**：合同中识别的供应商名称。
- **总体合规性**：合同是否总体符合 GDPR 要求。
- **总结说明**：对合同合规性的总结和建议。

```json
{
  "vendor_name": "Example Vendor",
  "overall_compliant": false,
  "summary_notes": "The contract with EFG, Inc. has been found to have noncompliant clauses. Specifically, the data transfer and subprocessor engagement clauses do not align with GDPR guidelines. It is recommended to review and update these clauses to ensure full compliance with GDPR regulations."
}
```

## 查看non-compliance结果：

[ClauseComplianceCheck(clause_text='Vendor may transfer data to any country where it maintains operations', matched_guideline=GuidelineMatch(guideline_text='A group of undertakings, or a group of enterprises engaged in a joint economic activity, should be able to make use of approved binding corporate rules for its international transfers from the Union to organisations within the same group of undertakings, or group of enterprises engaged in a joint economic activity, provided that such corporate rules include all essential principles and enforceable rights to ensure appropriate safeguards for transfers or categories of transfers of personal data.', similarity_score=0.7, relevance_explanation='The guideline discusses international transfers of data and the use of corporate rules to ensure appropriate safeguards, which is relevant to the clause allowing data transfers to any country where the vendor maintains operations.'), compliant=False, notes="The clause does not specify any safeguards or conditions for data transfers, which may not align with the guideline's recommendation for additional safeguards via contractual commitments. It is recommended to review and update the clause to ensure compliance with the guideline."), ClauseComplianceCheck(clause_text='Vendor may engage subprocessors without prior Client approval', matched_guideline=GuidelineMatch(guideline_text='Where a processor engages another processor for carrying out specific processing activities on behalf of the controller, the same data protection obligations as set out in the contract or other legal act between the controller and the processor shall be imposed on that other processor by way of a contract or other legal act under Union or Member State law, in particular providing sufficient guarantees to implement appropriate technical and organisational measures in such a manner that the processing will meet the requirements of this Regulation.', similarity_score=0.8, relevance_explanation='The guideline addresses the engagement of subprocessors and the need to impose data protection obligations on them, which is directly relevant to the clause.'), compliant=False, notes='The clause is not compliant with the guideline as it allows the vendor to engage subprocessors without prior client approval, which may not meet the data protection obligations and guarantees required by the regulation.')]
  
## 贡献与反馈

欢迎对该项目进行贡献和反馈！如果您有任何问题或建议，请通过 GitHub Issues 提交。

## 许可证

该项目采用 MIT 许可证。详情请参见项目根目录下的 `LICENSE` 文件。
