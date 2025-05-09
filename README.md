# LLM-based Text Summarization and Sentiment Analysis Pipeline

**Technologies:** Python, PyTorch, Transformers, Langdetect, ROUGE, DistilBERT, Kaggle

## **Introduction**
This project focuses on developing and evaluating an LLM-based approach for text summarization. The primary goal is to fine-tune pre-trained models on domain-specific data using pseudo-labels and apply sentiment analysis for identifying top reviews. The key points of my approach are as follows:

- I used the text summarization models: **Pegasus, MT5, and Flan-T5** and fine-tuned them on a domain-specific dataset (reviews) and leveraged pseudo-labels generated by **Facebook's BART** as the data itself did not contain any labels (i.e., the summarized reviews). The rationale behind this approach stems from the need to adapt large language models (LLMs) to specific tasks and domains, where pre-trained models often underperform due to a lack of domain-specific knowledge. By fine-tuning these models and using pseudo-labels, I tried to bridge the gap between general-purpose pre-trained models and domain-specific performance.

- I observed that some reviews were in Hindi, due to which I used **Langdetect** to translate the text to English.

- I used the **ROUGE score** to determine the efficacy of the models.
- I used **DistilBERT** for sentiment analysis to identify the top reviews.  

## **Understanding of Concepts**

### **Why Fine-Tuning?**
Pre-trained models like **Pegasus, MT5, and Flan-T5** are trained on massive, diverse datasets, making them excellent at general text summarization. However, they often struggle with domain-specific nuances, such as the language and structure of reviews. Fine-tuning allows these models to adapt to the specific vocabulary, tone, and context of reviews, improving their summarization quality.

### **Why Pseudo-Labels from BART?**
The provided data did not contain the labels (i.e., the summaries for the reviews), so to fine-tune the LLMs, I used the **knowledge distillation** technique, wherein **Facebook’s BART** acted as the teacher model by generating the pseudo-labels that were leveraged for training the student models: Pegasus, MT5, and Flan-T5. Knowledge distillation was necessary here due to a lack of human-annotated labels in the reviews dataset.

### **Why ROUGE score for text summarization evaluation?**
**ROUGE (Recall-Oriented Understudy for Gisting Evaluation)** is a widely used metric for evaluating text summarization models by measuring the overlap of n-grams, sequences, and sentence structures between generated and reference summaries. It is particularly useful for **abstractive summarization** as it captures content preservation while allowing some variation in phrasing. 
- **ROUGE-1:** Unigram overlap  
- **ROUGE-2:** Bigram overlap  
- **ROUGE-L:** Longest common subsequence  
These provide a comprehensive assessment of summary quality.

### **Why DistilBERT for Sentiment Analysis?**
**DistilBERT** is a lightweight, efficient version of **BERT**, making it ideal for sentiment analysis tasks where computational resources and speed are critical. I used DistilBERT to identify both the **top 5 negative reviews** as well as the **top 5 positive reviews**.

## **Code Description**

### **1. Fine-Tuning Process**
- The fine-tuning process was designed to be **modular and reproducible**.
- Each model (**Pegasus, MT5, Flan-T5**) was fine-tuned using the same pipeline to ensure consistency.
- The code was structured to handle:
  - Data loading
  - Preprocessing
  - Model training
  - Evaluation separately
- **Efficiency** was prioritized by using **mixed-precision training** and leveraging **GPU acceleration** on Kaggle, reducing training time without compromising model performance.

### **2. Pseudo-Label Generation**
- The pseudo-label generation pipeline was optimized for **scalability**. 
- **BART** was used in an **inference-only mode**, ensuring that the process was computationally efficient.
- The generated pseudo-labels were validated using a small manually annotated dataset to ensure their quality before being used for fine-tuning.

### **3. Sentiment Analysis with DistilBERT**
- The sentiment analysis module was designed to be **lightweight and fast**.
- **DistilBERT's smaller size** allowed for quick inference, making it suitable for processing large volumes of reviews.

## **Evaluation Strategy and Observations**

### **Comparing Fine-Tuned vs. Pre-Trained Models**
The evaluation strategy focused on comparing the performance of **fine-tuned models** against their **pre-trained counterparts**. 
- **ROUGE score (ROUGE-1, ROUGE-2, ROUGE-L)** was used to measure summarization quality. 

### **Observation**
- The **fine-tuned Pegasus** outperformed the pre-trained Pegasus, while it was **vice-versa** for both MT5 and Flan-T5, where the pre-trained models performed better than the fine-tuned versions. 
- This is because **Pegasus** is already optimized for text summarization, while **MT5** and **Flan-T5** are more general-purpose and require larger, more diverse datasets for effective fine-tuning.

## **Analysis and Insights**

### **1. Why Fine-Tuned Pegasus Performed Better**
- **Pegasus** is specifically optimized for **text summarization**, making it naturally more efficient in this task.  
- The **fine-tuned Pegasus** benefited from exposure to domain-specific data and pseudo-labels, enabling it to better handle:
  - Informal language  
  - Product-specific jargon  
  - Varying review lengths  
- In contrast, **MT5 and Flan-T5** performed worse when fine-tuned, likely due to their general-purpose nature. They would require a larger and more diverse review dataset to fine-tune effectively.
- The **pseudo-labels generated by BART** acted as a form of "soft supervision," guiding the models to produce summaries more aligned with the desired output.

### **2. Challenges and Limitations**
- Ensuring the quality of **pseudo-labels** was a challenge. While **BART** is powerful, its summaries were not always perfect and occasionally introduced **noise** into the training data.
- The **computational cost** of fine-tuning multiple models was significant, even with efficient techniques.
- The dataset was **limited and multilingual**. More data could significantly improve the fine-tuning performance, especially for **MT5 and Flan-T5**.

### **3. Insights from Sentiment Analysis**
- **DistilBERT's** efficiency made it well-suited for large-scale sentiment analysis. 
- It accurately classified sentiment, helping identify the most impactful reviews (i.e., the **top 5 positive** and **top 5 negative** reviews).

## **Potential Improvements**

### **1. Improving Pseudo-Label Quality**
- Future work could explore using **ensemble methods** to generate pseudo-labels, combining outputs from multiple models (e.g., BART, T5) to produce higher-quality and more diverse pseudo-labels.
- Instead of using simple knowledge distillation, exploring **distillation-step-by-step** (link: [Distillation Step-by-Step](https://research.google/blog/distilling-step-by-step-outperforming-larger-language-models-with-less-training-data-and-smaller-model-sizes)) could be beneficial. This technique generates **rationales behind the output**, helping the student model learn more effectively.
- **GPT-4** or other LLMs could be used to generate higher-quality pseudo-labels.

### **2. Enhancing Sentiment Analysis**
- While **DistilBERT** was effective, more advanced techniques such as **Aspect-Based Sentiment Analysis (ABSA)** could provide deeper insights into specific review aspects (e.g., product features, customer service).

### **3. Scaling to Larger Datasets**
- The current approach was tested on a **small review dataset**.
- Scaling to larger datasets (e.g., millions of reviews) would require:
  - **Distributed training**
  - More efficient data pipelines

### **4. Real-Time Applications**
- The fine-tuned models could be deployed in **real-time applications**, such as e-commerce platforms, to provide **instant review summaries**.
- This would require further optimization for **latency and throughput**.
- **Dynamic Sentiment Tracking:** The sentiment analysis module could be used to track sentiment changes over time (e.g., after product updates or marketing campaigns).

## **Conclusion**
Through this project, I successfully demonstrated the effectiveness of **fine-tuning Pegasus, MT5, and Flan-T5** using **pseudo-labels generated by BART**, with **DistilBERT** enabling the identification of **top reviews** through sentiment analysis.
