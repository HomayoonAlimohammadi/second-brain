#ai #embedding #machine_learning #neural_networks #deep_learning #word_embedding #nlp #natural_language_processing 

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

### Reference:
https://www.youtube.com/watch?v=viZrOnJclY0&ab_channel=StatQuestwithJoshStarmer
