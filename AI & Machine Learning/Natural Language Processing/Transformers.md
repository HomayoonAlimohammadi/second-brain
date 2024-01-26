#ai #natural_language_processing #nlp #neural_networks #transformers #attention #machine_learning 

![[Attention is all you need.pdf]]
## Part 1:

![[transformers.webp]]


1. **Introduction to Transformers:**
   - Transformers have revolutionized NLP in recent years.
   - Introduced in the paper "Attention is all you need."
   - Outperform existing models in text data applications.
   - Examples include Google’s BERT and OpenAI’s GPT series.

2. **Course Structure:**
   - The course covers Transformers' basics, architecture, and internal workings.
   - Focus on multi-head attention, a key component of Transformers.
   - Additional topics include Beam Search and Bleu Score in NLP.

3. **Transformer Architecture:**
   - Consists of Encoder and Decoder stacks with Embedding layers.
   - Each stack has multiple identical layers.
   - Encoders focus on self-attention and feed-forward layers.
   - Decoders include self-attention, feed-forward, and Encoder-Decoder attention layers.
   - Residual connections and LayerNorm layers are also present.

4. **Attention Mechanism:**
   - Central to Transformer’s performance.
   - Allows the model to focus on relevant words in the input.
   - Self-attention relates each word in the sequence to every other word.
   - Multiple attention scores for each word capture nuances and semantics.

5. **Training Transformers:**
   - Involves input and target sequences.
   - Uses embeddings with position encoding.
   - The output layer generates word probabilities.
   - Loss function and back-propagation are used for training.

6. **Inference in Transformers:**
   - Only the input sequence is available.
   - Output is generated in a loop, feeding the output sequence back to the Decoder.
   - Uses start-of-sentence and end-of-sentence tokens.

7. **Teacher Forcing:**
   - A training technique where the target sequence is fed to the Decoder.
   - Prevents compounding errors and speeds up training.

8. **Applications of Transformers:**
   - Used in various NLP tasks like machine translation, text summarization, question-answering, etc.
   - Different architectures for different applications (e.g., Sentiment Analysis, Language Models).

9. **Comparison with RNNs:**
   - Transformers replaced RNNs, LSTMs, and GRUs in NLP.
   - Overcome limitations of RNNs like handling long-range dependencies and sequential processing.
   - Process all words in parallel, enabling faster computation.

10. **Further Learning:**
   - The course will delve deeper into the internal functionality of Transformers.
   - Other related series include Audio Deep Learning, Geolocation Machine Learning, and Image Caption architectures.

## Part 2:

![[transformer2.webp]]
![[attention_block.webp]]
![[attention_score.webp]]


1. **Transformer Architecture Components:**
   - **Data Inputs:** Both Encoder and Decoder have an Embedding layer and a Position Encoding layer.
   - **Encoder Stack:** Contains multiple Encoders, each with a Multi-Head Attention layer and a Feed-forward layer.
   - **Decoder Stack:** Comprises several Decoders, each with two Multi-Head Attention layers and a Feed-forward layer.
   - **Output:** Includes a Linear layer and a Softmax layer.

2. **Embedding and Position Encoding:**
   - **Embedding Layer:** Encodes the meaning of each word.
   - **Position Encoding Layer:** Represents the position of each word in the sequence.
   - **Combination:** The Transformer adds these two encodings together.

3. **Matrix Dimensions:**
   - Operates on matrices representing a batch of sequence samples.
   - Embedding takes a matrix of word IDs and encodes each into a word vector.
   - Position Encoding produces a similarly shaped matrix to the embedding matrix.

4. **Encoder and Decoder Stacks:**
   - **Encoder:** Receives input from the Embedding and Position Encoding. Each Encoder passes its input through a Self-attention layer and a Feed-forward layer.
   - **Decoder:** Similar to the Encoder but includes an additional Encoder-Decoder attention layer. Receives input from the Output Embedding and Position Encoding.

5. **Attention Mechanism:**
   - Used in three places: Self-attention in both the Encoder and Decoder, and Encoder-Decoder-attention in the Decoder.
   - Attention layer takes three parameters: Query, Key, and Value.
   - In Self-attention, the layer's input is passed to all three parameters.
   - In Encoder-Decoder attention, the output of the final Encoder is passed to the Value and Key parameters.

6. **Multi-head Attention:**
   - Involves repeating the Attention processor several times in parallel.
   - Enhances the Attention mechanism's power of discrimination.

7. **Attention Masks:**
   - Used to zero attention outputs where there is padding in input sentences.
   - Prevents the Decoder from 'peeking' ahead at the rest of the target sentence during prediction.

8. **Generating Output:**
   - The last Decoder's output is passed to the Output component.
   - Linear layer projects Decoder vector into Word Scores for each unique word.
   - Softmax layer converts scores into probabilities.

9. **Training and Loss Function:**
   - Uses cross-entropy loss to compare the generated output probability distribution to the target sequence.
   - Loss is used to compute gradients for training the Transformer via backpropagation.

10. **Conclusion:**
   - Provides an understanding of the Transformer's functionality during Training.
   - Highlights the importance of the Multi-head Attention module.
   - Future articles will delve deeper into the details of Attention computation.

### references:
- Part 1: https://towardsdatascience.com/transformers-explained-visually-part-1-overview-of-functionality-95a6dd460452
- Part 2: https://towardsdatascience.com/transformers-explained-visually-part-2-how-it-works-step-by-step-b49fa4a64f34