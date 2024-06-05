# Done

[ArXiV Draft](Done%20b715c92198314529880806d9f206803d/ArXiV%20Draft%2035c75a5f1929460fb199edd5fce9a6fe.md)

[NAACL and ACL](Done%20b715c92198314529880806d9f206803d/NAACL%20and%20ACL%20cff04ab3612d474cbca60c48f2967762.md)

Interval Sequences on Larger Models - Explora tests (incr num)

Run Llama-2 on same data by March 22nd: [https://aclrollingreview.org/dates](https://aclrollingreview.org/dates)

March 24th - 27th is Author Response for ACL

- ⚠️ In new drive folder, test prompts on Llama-2. See if can do 2, 4, 6 and Fibonacci
    
    [test_prompts_llama2_7b.ipynb](https://colab.research.google.com/drive/1zFWHdQkE8vpRe6PcQwoOVz1qAdNBwtdY#scrollTo=DcZG9rm2IAiA)
    
    - issue: why two ranking lists? first is for empty string.
- ✅ [Llama2_num_runModel_explrTests_v1.ipynb](https://colab.research.google.com/drive/143roK4-FGgqqxtoM1ohDRe95enEdcVW6#scrollTo=vKYgaZ9JjihZ)
    - ✅ try get unablated (orig) logit diff score
        - ✅ ISSUE: freezes when tokenizing dataset for `dataset = Dataset(prompts_list, pos_dict, model.tokenizer)` because of long list of `Keyword arguments {'add_special_tokens': False} not recognized.`
            - search this repo for why Keyword arguments {'add_special_tokens': False} not recognized. occurs when llama 2 uses model.tokenizer: [https://github.com/neelnanda-io/TransformerLens](https://github.com/neelnanda-io/TransformerLens)
                
                The issue you're experiencing with the `{'add_special_tokens': False}` keyword argument not being recognized appears to be related to how different tokenizers handle the addition of special tokens (such as beginning-of-string tokens) to inputs. Specifically, in the context of the TransformerLens project, there have been discussions and fixes regarding how the OPT and LLaMA models' tokenizers prepend tokens to the inputs.
                
                For the Facebook OPT tokenizer, it was observed that it automatically prepends a `</s>` token to OPT prompts unless the tokenizer call explicitly sets `add_special_tokens=False`. This behavior was not affecting the GPT tokenizer. A solution proposed was to conditionally set `add_special_tokens=False` based on the model type, specifically for OPT models, to prevent this automatic prepending from happening【5†source】.
                
                Further discussion on this topic suggested that handling of the `add_special_tokens` parameter might need to be more nuanced, potentially involving checks against the model's tokenizer name or the model architecture to ensure compatibility across different models without introducing unexpected bugs【5†source】【6†source】.
                
                Another related issue was reported for LLaMA models, where the beginning-of-string (bos) token was always prepended for strings processed by the `HookedTransformer` methods, which use the `.to_tokens()` function. This was due to the `add_special_tokens=False` setting not being applied to LLaMA models, although LLaMA's tokenizer, similar to OPT's, adds a bos token by default. A fix was proposed to automatically infer the `add_special_tokens` parameter for the tokenizer, addressing the issue where an extra bos token was being added unintentionally【7†source】.
                
                These discussions and fixes illustrate the importance of carefully managing tokenizer settings and the `add_special_tokens` parameter when working with different models within the TransformerLens project or similar frameworks. The goal is to ensure that tokenization behavior aligns with the expectations for each specific model type, preventing issues like unwanted token prepending.
                
            
            SOLN: For now, just use dataset of 12 samples %%capture
            
        - ✅ ISSUE: orig score is 0
            - ✅ Check corr logits and incorr logits are diff
                
                They’re the same ids
                
            - Check prompts dict for what was in corr and incorr
                
                ![Untitled](Done%20b715c92198314529880806d9f206803d/Untitled.png)
                
                The logit diff isn’t measured by the 1st rank token minus incorr token. It’s what’s marked as the correct token, regardless of if correct token is 1st ranked or not. This means it doesn’t matter what 
                
                SOLN: In dataset, just take `self.tokenizer.encode(prompt["corr"])[-1]` instead of `0` because `model.tokenizer.decode(29945)` is 5
                
    - ✅ ISSUE: out of mem even for 12 samps
        
        decide to try diff route
        
    - ✅ Try pure seq instead for 12 samps
        - ISSUE: the incorr token logits are higher than corr token logits (neg diff score)
            - Check token ids of prompts and corr, incorr.
                
                Notice we use up to 14. The padding for llama’s tokenzier on numbers after 10 seeems to split them up, meaning there is more padding on “1 2 3 4” earlier prompts. Does this make a difference? Yes; putting it in with extra padding messes it up, not predicting “5”. 
                
- ✅ if you add padding to the end of a prompt, will transformers treat the padding as different than if there was no padding, and predict for the padding as the last token instead?
    
    Yes, when you add padding to the end of a prompt for a transformer model, it can affect the model's predictions, but perhaps not in the way you might initially think. Padding tokens are a special kind of token used to bring sequences of words or tokens to a uniform length, enabling batch processing of input data by the model. These padding tokens are typically ignored by the model during predictions due to the use of attention masks.
    
    In the context of natural language processing (NLP) and transformer models, an attention mask is applied to specifically ignore these padding tokens during the attention calculations. The attention mechanism is a core component of transformer models, allowing them to focus on relevant parts of the input data when making predictions. By using an attention mask, the model can differentiate between actual input data and padding, effectively treating the padding as non-existent for the purposes of prediction.
    
    So, while the presence of padding alters the input sequence, the model is designed to predict ignoring these padding tokens as if they were not part of the input. This means that the model will not "predict for the padding as the last token" because, from the model's perspective, the padding tokens are masked out and do not influence the prediction of actual content tokens.
    
    However, the addition of padding can still have indirect effects on model performance, especially if the padded sequences are significantly longer than the original content, potentially impacting computational efficiency. But as far as predictions go, transformer models are designed to handle padding gracefully and predict as if the padding were not there, thanks to attention masking.
    
- ✅ Try for only 1 sample to avoid inconsistent padding issues
    
    [Llama2_num_runModel_explrTests_v2.ipynb](https://colab.research.google.com/drive/1mx0zSN7X5u5SBHZR5MrMEVFS-fSD4s36)
    
    - ✅ ISSUE: orig score still neg, same values for first prompt, despite `utils.test_prompt` now correctly predicting 5 (example prompt has no padding at end)
        
        So padding wasn’t the issue
        
        - Look at highest tokens to see what it’s predicting then
            
            SOLN:
            
            ![Untitled](Done%20b715c92198314529880806d9f206803d/Untitled%201.png)
            
            Either:
            
            1. It’s predicting using the empty space as the prompt (not likely bc/ then all prompts would be the same prediction, but this isn’t happening)
            2. Or, it uses the prompt as input but will predict empty space
        
    - ✅ [Give space in prompt end](https://colab.research.google.com/drive/1mx0zSN7X5u5SBHZR5MrMEVFS-fSD4s36#scrollTo=lAmonB77JUrG&line=1&uniqifier=1) for utils
        
        ![Untitled](Done%20b715c92198314529880806d9f206803d/Untitled%202.png)
        
    - ✅ Give space in prompt end to get logits
        
        It works
        
- ✅ post on ARENA slack your findings
    
    It seems that when I tried a prompt like "1 2 3 4", the llama model in transformerlens, the tokens with the highest logit would be the "" empty space, and the top 5 wouldn't have "5" either. But using "1 2 3 4 " with a space at the end allowed 5 to be the token with the highest logit. Still looking for a better work around as this seems gimmicky. Also, seem to be running of memory easily with A100 for llama-2 so may switch to nnsight as I did before.
    
- ✅ run ‘ablate each attention head’ on 1 samp of llama-2
    
    Llama-2 has 32 heads per layer
    
    - ✅ ISSUE: S0 lookup issue because pos_dict starts using S1 hard-coded, so change to S0 to S3 instead in both prompts_lsit and Pos_dict and prompst_list_2
    - ✅ ISSUE: Now the loop tries to find S4, saying it’s not found. So look in code to see why it’s looking for S4 when neither prompts_list nor pos_dict have it as a key.
        
        ```python
            SEQ_POS_TO_KEEP = {}
            for i in range(len(model.tokenizer.tokenize(dataset_2.prompts[0]['text']))):
                CIRCUIT['S'+str(i)] = lst
                if i == len(model.tokenizer.tokenize(dataset_2.prompts[0]['text'])) - 1:
                    SEQ_POS_TO_KEEP['S'+str(i)] = 'end'
                else:
                    SEQ_POS_TO_KEEP['S'+str(i)] = 'S'+str(i)
        ```
        
        SOLN: Running this in colab, we find there’s 8 positions because the random prompt using integers greater than 10, which breaks down into tokens of more than 1.
        
    - ISSUE: it’s NOT the integer, it’s tokenizer the empty space as a token!
        
        ![Untitled](Done%20b715c92198314529880806d9f206803d/Untitled%203.png)
        
        Seems like the llama tokenizer.tokenize works differently than the tokenizer for GPT-2, which would break ‘1 4 4 3’ into just four tokens for each integer instead of making each space be a token
        
- ⚠️ pos_dict all pos (even spaces)
    
    [Llama2_num_runModel_explrTests_v3.ipynb](https://colab.research.google.com/drive/1w49rGrjH55D2pBwqtH_qS7kd10xzuT52#scrollTo=vKYgaZ9JjihZ)
    
    This deletes previous explora tests and cleans up code. Don’t hard code pos dict but make key for every tokenized token
    
    - ⚠️ ISSUE: why are the scores all negative (meaning removal makes incorr be higher than corr for any of these components, so they’re essential?)
- ✅ try fix tokenizer using slack suggestion
    
    [Llama2_num_runModel_explrTests_v4.ipynb](https://colab.research.google.com/drive/1W-w3CqxNKN08VUwJblLy6g1vvOeG7Lva)
    
    `tokenizer = LlamaTokenizer.from_pretrained(LLAMA_2_7B_CHAT_PATH, use_fast= False, add_prefix_space= False)` 
    
    > using `'use_fast'= False, 'add_prefix_space'= False` when defining the tokenizer makes the problem go away (though you have to add spaces manually. it's a bit messy
    > 
    
    This doesn’t fix test prompt. Getting rid of space at end also makes it predict ‘ ‘ instead of corr token.
    
- ✅ respond to slack about persisting issue
    
    I added them using tokenizer = LlamaTokenizer.from_pretrained(LLAMA_2_7B_CHAT_PATH, use_fast= False, add_prefix_space= False) and it fixes encode and decode (encode('5') encodes it into padding <s> and 5 IDs without the ' ') but it seems test_prompt still has two rankings for ' ' and 5 (it's not directly used in ablation so it doesn't need to be correct) and logits_original = model(dataset.toks) still predicts empty space ' ' as the top token unless the space is added to the end of the input prompt (I suppose this is what you meant by add spaces manually)? 
    
- ✅ Llama2_num_runModel_explrTests_v3: fix why are the scores all negative
    
    SOLN:  You changd the wrong line of code of what to ablate. 
    
    ```
    circ = [(layer, head) for layer in range(32) for head in range(29, 31)]
    to_loop = circ.copy()
    ```
    
    `circ` should ALWAYS start off as the full circuit, while to_loop should contain which heads to remove from circ one by one but independent of one another (this isn’t acdc yet)!
    
    Try going from 20 to 30 first, as most likely that’s what’s impt
    
- ⚠️ run on 1…12 pure num samps
    
    [Llama2_num_runModel_explrTests_v5.ipynb](https://colab.research.google.com/drive/11Nmwec2lwDDakMskq-ENo0HtjDXliOyx#scrollTo=vKYgaZ9JjihZ)
    
    Before, was only on 1 samp
    
    - ISSUE: tokenizer breaks up digits into individual tokens
        
        ![Untitled](Done%20b715c92198314529880806d9f206803d/Untitled%204.png)
        
        It appears to be a **Character-Level Tokenization, even though this says it’s BPE:**
        
        [https://huggingface.co/docs/transformers/en/model_doc/llama2](https://huggingface.co/docs/transformers/en/model_doc/llama2)
        
- ✅ run on  1…9 pure num samps (5 total)
    
    [Llama2_num_runModel_explrTests_v6](https://colab.research.google.com/drive/1_X-FGghmCEAQzbKIagMOCuiVl7Ew_Rl4#scrollTo=DcZG9rm2IAiA)
    
- ✅ [Llama2_numerals_amongItems_AttnPats.ipynb](https://colab.research.google.com/drive/1iiTVFV7bw-FQBmt22W_4sz6epni36BE3)
    
    uses transformerlens because saves the cache
    
    Llama-2 NEEDS A100 high-ram just to load; V100 won’t work
    
    - why A100 has more memory than V100
        
        The NVIDIA A100 and V100 are both high-performance GPUs designed for data center use, catering to different demands in computation, AI, and deep learning tasks. The A100, being a more recent addition to NVIDIA's lineup, boasts more memory than the V100 due to several factors:
        
        1. **Technological Advancements**: Since the A100 is newer, it benefits from advancements in semiconductor and memory technologies, allowing for higher memory capacities within the same physical space constraints.
        2. **Increased Data Demands**: Modern AI and machine learning workloads have grown in complexity and size, necessitating GPUs with more memory to efficiently process and analyze large datasets.
        3. **Architecture Improvements**: The A100 utilizes the NVIDIA Ampere architecture, which offers significant improvements over the previous Volta architecture used in the V100. These improvements include better memory management and higher efficiency, making it feasible to include more memory.
        4. **Market Expectations**: As the industry progresses, there's a continuous push for more powerful computing resources. NVIDIA designed the A100 to meet these growing demands, ensuring it remains competitive by including more memory to handle larger and more complex workloads.
        5. **Use Case Flexibility**: With more memory, the A100 can cater to a wider range of applications, from AI training and inference to high-performance computing (HPC) tasks. This flexibility makes it a more versatile and attractive option for data centers that handle diverse workloads.
        
        The A100 doesn't just have more memory; it also includes several architectural enhancements over the V100 that improve its overall performance and efficiency, further justifying its place as the successor in NVIDIA's GPU lineup.
        
    
    Use attn heads from here: [https://colab.research.google.com/drive/1_X-FGghmCEAQzbKIagMOCuiVl7Ew_Rl4#scrollTo=susSZdqpqVzd&line=7&uniqifier=1](https://colab.research.google.com/drive/1_X-FGghmCEAQzbKIagMOCuiVl7Ew_Rl4#scrollTo=susSZdqpqVzd&line=7&uniqifier=1)
    
    Found seq detection heads: (3, 24), (5, 25)
    
    analogue to 7.11: (16, 0)
    
    and successor heads: (20, 17) - same one from succ head paper? The paper doesn’t mention what specific one from llama-2 it found, just it found multiple
    
    > In the rest of this work, we perform a case study on the attention head (L12H0) with the maximal successor score in Pythia-1.4B.
    > 

Interval Sequences on Larger Models - Fibonacci

- ✅ [Fibonacci plan](../Fibonacci%20plan%202e0e51805bee4b44ba09fe671e89ab19.md)
- 🐣 [[Llama2_fibonacci.ipynb](https://colab.research.google.com/drive/1oFGRQxRdqcXb9NDP6Mp-3t6M3LouvWe-#scrollTo=vKYgaZ9JjihZ)](../Expm%20Results-%20NAACL%208de8fe5b943641ec92c4496843189d36/Expm%20Results-%20Llama%202%20bc7eb66db28f4bbda27403862beea6ec/Llama2_fibonacci%20ipynb%20b59d5b5e54564c2f86a90f04761c6dec.md)
    - ✅ ISSUE: fibo reqs above 10, but tokenizer breaks double digits into single chars
        
        SOLN: make pos_dict INSIDE Dataset(), after creating `self.toks` because that uses padding=True, which makes all prompts in the tensor have the same length using padding:
        
        ```
        pos_dict = {}
          for i in range(len(self.toks)):
              pos_dict['S'+str(i)] = i
        ```
        
    - ⚠️ ISSUE: must still have same keys for all prompts due to patching requiring specific keys in clean needing the corresponding key from corrupted (?)
        
        ```jsx
             70         for (mlp_type, layer_list) in circuit.items():
             71             seq_pos = seq_pos_to_keep[mlp_type]
        ---> 72             indices = means_dataset.word_idx[seq_pos]
        ```
        
        TRY: start from double digits, don’t go into triple digits
        
        ISSUE: fibonacci quickly becomes triple digits
        
        TRY: clean prompts are double digits at most, corrupted prompts ONLY use double digits
        
        ISSUE: the predicted token, if double digit, may only be a single token! (eg. 89 is 8)
        
        ![Untitled](Done%20b715c92198314529880806d9f206803d/Untitled%205.png)
        
    - ✅ Ask DQ on slack
        
        Hi, I was looking to see what's the best way to get the logits for an answer that is split into two tokens? I'm looking to get logits_original = model(dataset.toks) to measure logit diff between clean and corrupted for say "13", but llama-2 splits that into two tokens (as in the image attached). Was thinking of doing "1 2 3 5 8 1" to predict "3" but that seems to be giving too much info to the model. Was also thinking about a way to feed the 1 back into the model and measuring its output logit value of "3", not sure if that is a good approach.
        
    - ✅ David Quarrel response
        
        By definition, p(x,y) = p(x|y)p(y). So, I would measure the probability of "1 2 3 5 8" -> 1 and "1 2 3 5 8 1" -> 3, and then multiply
        
        so yeah, two forward passes
        
        Wait, I'm silly. If you give it "1 2 3 5 8 1", then the output is of shape `(1,6,vocab_size)` . The last entry represents the distribution P( . | 1 2 3 5 8 1), and the second to last represents P( . | 1 2 3 5 8 )
        So, you can extract both out, and then multiply (though it's better to work with log probabilities and add for numerical stability
        You may also want to look into the `logsumexp` function provided by torch, also for numerical stability purposes
        
- ✅ test if tokenizer really is char level, or just for double digits
    
    ![Untitled](Done%20b715c92198314529880806d9f206803d/Untitled%206.png)
    
    It’s not; just for double digits
    

Observe Mathematical Reasoning Effects

- ✅ Plan updates
    
    [https://docs.google.com/document/d/1sBKNJ7Xut0kPnFGqRfwn0LqBoiJrgf3_4S3zNevomAQ/edit](https://docs.google.com/document/d/1sBKNJ7Xut0kPnFGqRfwn0LqBoiJrgf3_4S3zNevomAQ/edit)
    
- 🐣 TransformerLens: Get code to generate math reasoning (not just next logit)
    
    [ablate_seqcont_then_math.ipynb](https://colab.research.google.com/drive/1OVMkA1IZKZLmKq2paGRIXykzm5fxrGIx)
    
    - ✅ 🐣 ISSUE: after adding hook, cannot do this:
        
        ```jsx
        for i in range(10):
            print(f"{tokens.shape[-1]+1}th char = {next_char!r}")
            # Define new input sequence, by appending the previously generated token
            tokens = t.cat([tokens, next_token[None, None]], dim=-1)
            # Pass our new sequence through the model, to get new output
            logits = model(tokens)
        ```
        
        `RuntimeError: The size of tensor a (4) must match the size of tensor b (36) at non-singleton dimension 1`
        
        EXPLAIN: This is related to sequence length. The ablation ablates at fixed positions (4), but the tokens passed in have seqlen 36
        
        QUESTION 1: But it takes an input  `'1 2 3'` of length 3 instead of 4?
        
        QUESTION 2: But this code can only take an input of length 2. 
        
        ```jsx
        example_prompt = "1 2"
        example_answer = " 3"
        utils.test_prompt(example_prompt, example_answer, model, prepend_bos=True)
        ```
        
    - ✅ 🐣 ISSUE: after ablating  all heads but not MLPs, ARENA approach to generate tokens yields correct token, but transformerlens approach to predict prompt yields wrong token
        
        DETAILS: This approach seems unaffected by ablation
        
        - code
            
            ```
            reference_text = '1 2 3'
            tokens = model.to_tokens(reference_text).to(device)
            
            print(f"Sequence so far: {model.to_string(tokens)[0]!r}")
            
            for i in range(10):
                print(f"{tokens.shape[-1]+1}th char = {next_char!r}")
                # Define new input sequence, by appending the previously generated token
                # tokens = t.cat([tokens, next_token[None, None]], dim=-1)
                # Pass our new sequence through the model, to get new output
                logits = model(tokens)
                # Get the predicted token at the end of our sequence
                next_token = logits[0, -1].argmax(dim=-1)
                # Decode and print the result
                next_char = model.to_string(next_token)
            ```
            
            ![Untitled](Done%20b715c92198314529880806d9f206803d/Untitled%207.png)
            
        
        But this approach does seem affected
        
        - code
            
            ```jsx
            example_prompt = "1 2"
            example_answer = " 3"
            utils.test_prompt(example_prompt, example_answer, model, prepend_bos=True)
            ```
            
            ![Untitled](Done%20b715c92198314529880806d9f206803d/Untitled%208.png)
            
        
        We check this by resetting hooks and looking at difference. The first is unchagned after reset hooks, but the second gets the correct next token as ‘3’
        
        TRY: check if logits of ARENA appraoch truly do match before and after ablating all components
        
        RESULT: the logits don’t match before and after ablate
        
        HYPOTHESIZE: the ablation DOES work, but ‘1 2 3’ still is strong enough to get ‘4’ due to the MLPs, just the ranking is much lower. But ‘1 2’, with less, is not enough. We would need to figure out the issue above with why it can only take prompts of a certain length after hook ablation to compare ‘1 2 3’ using both ARENA and TL approaches
        
    - ✅ 🐣 Go back to “[ISSUE: after adding hook](../Project%20Planning%20(quests)%203798a71e7c5d4a888cad9a7d25a1275c.md),” and study how ablation code works to find 1) why prompts of different lengths cannot be put in, 2) why is it not len 4 prompts that are allowed, but ARENA approach is len 3 and TL utils example prompt is len 2
        - ✅ Comment ablation functions
        - ✅ 🐣 Look at traceback error to find where in ablation functions error occurs
            
            ```jsx
            <ipython-input-51-36366ada469c> in hook_func_mask_head(z, hook, components_to_keep, means)
                 63     '''
                 64     mask_for_this_layer = components_to_keep[hook.layer()].unsqueeze(-1).to(z.device)
            ---> 65     z = t.where(mask_for_this_layer, z, means[hook.layer()])
                 66 
                 67     return z
            
            RuntimeError: The size of tensor a (4) must match the size of tensor b (5) at non-singleton dimension 1
            ```
            
            - how does this work in detail:
            `z = t.where(mask_for_this_layer, z, means[hook.layer()])`
                
                ### Components of the Line
                
                1. **`t.where(condition, x, y)`**:
                    - This is a PyTorch function that returns a tensor of elements selected from either `x` or `y`, depending on the `condition`.
                    - For each element in the `condition` tensor:
                        - If the condition is `True`, the corresponding element from `x` is taken.
                        - If the condition is `False`, the corresponding element from `y` is taken.
                2. **`mask_for_this_layer`**:
                    - This is a boolean tensor of shape `(batch, seq, head, 1)`.
                    - It indicates which elements of `z` should be kept (`True`) and which should be replaced (`False`).
                3. **`z`**:
                    - This is a tensor of ORIGINAL activations from the model, with shape `(batch, seq, head, d_head)`.
                    - It represents the output of the current layer's attention heads **before modification.**
                4. **`means[hook.layer()]`**:
                    - This is a tensor of the mean activations for the current layer, with the same shape as `z` but excluding the batch dimension, so it must be broadcastable to `(batch, seq, head, d_head)`.
                    - It represents the average activations that should be used to replace the original activations where the mask is `False`.
                
                ### What Happens in Detail
                
                1. **Broadcasting**:
                    - The `means[hook.layer()]` tensor, which has shape `(batch, seq, head, d_head)`, is broadcast to match the shape of `z`. Broadcasting in PyTorch automatically expands the smaller tensor along its dimensions to match the larger tensor's shape.
                2. **Condition Evaluation**:
                    - The `t.where` function evaluates the `mask_for_this_layer` condition for each element in the tensor.
                    - This mask indicates which elements of `z` should remain unchanged (`True`) and which should be replaced with the corresponding element from `means[hook.layer()]` (`False`).
                3. **Element-wise Selection**:
                    - For each element in `z`:
                        - If the corresponding element in `mask_for_this_layer` is `True`, the element from `z` is kept.
                        - If the corresponding element in `mask_for_this_layer` is `False`, the element from `means[hook.layer()]` is used instead.
                
                ### Example
                
                Let's consider an example to illustrate the process. Suppose we have:
                
                - `z` is a tensor with shape `(2, 3, 4, 5)` representing the activations.
                - `mask_for_this_layer` is a boolean tensor with shape `(2, 3, 4, 1)`.
                - `means[hook.layer()]` is a tensor with shape `(2, 3, 4, 5)` representing the mean activations.
                
                ```python
                import torch as t
                
                # Example tensors
                z = t.randn(2, 3, 4, 5)
                mask_for_this_layer = t.tensor([
                    [[[True], [False], [True], [False]],
                     [[True], [True], [False], [False]],
                     [[False], [True], [True], [False]]],
                
                    [[[False], [False], [True], [True]],
                     [[True], [False], [False], [True]],
                     [[True], [True], [True], [False]]]
                ])
                means_for_layer = t.zeros(2, 3, 4, 5)  # Example mean activations tensor
                
                # Applying t.where
                result = t.where(mask_for_this_layer, z, means_for_layer)
                
                ```
                
                In this example:
                
                - The `result` tensor will have elements from `z` where `mask_for_this_layer` is `True` and elements from `means_for_layer` where `mask_for_this_layer` is `False`.
                
                ### Summary
                
                The line `z = t.where(mask_for_this_layer, z, means[hook.layer()])` effectively replaces certain activations in `z` with their mean values from `means[hook.layer()]`, based on the boolean mask `mask_for_this_layer`. This allows selective modification of the model's activations, preserving important components and replacing the rest with a baseline value.
                
            - z is the ORIGINAL activations obtained by passing `tokens` through the model. The hook function gets z at run-time, so not a variable within the function. `means` is obtained by running the means dataset through the model, so its number of tokens per prompt must match the current len of  `tokens`
            - what is a referring to and what is b refeerring to
                - `a` refers to `z`
                - `b` refers to `means[hook.layer()]`
        - ✅ compare number of tokens in dataset (used to ablate) vs tokens that pass in from ARENA approach to generate text
            
            ![Untitled](Done%20b715c92198314529880806d9f206803d/Untitled%209.png)
            
            `model.to_tokens()` adds on a space in front
            
        - ✅ compare number of tokens that pass in from ARENA approach to generate text vs tokens used by `utils.test_prompt()`
            
            SOLN: `util.test_prompt()` used `prepend_bos=True`, so it only took in len 2. But now, is the same as ARENA approach when using `prepend_bos=False`
            
            ```
            example_prompt = "1 2 3"
            example_answer = " 3"
            utils.test_prompt(example_prompt, example_answer, model, prepend_bos=False)
            ```
            
        - ✅ how do I use this without appending an end text token at the front:
        `tokens = model.to_tokens(reference_text).to(device)`
            - ✅ how to get rid of first entry here: tensor([[50256,    16,   362,   513,   604]])
                
                `tokens_trimmed = tokens[:, 1:]`
                
            
            Actual SOLN: [https://www.notion.so/wlg1/Project-Planning-3798a71e7c5d4a888cad9a7d25a1275c?pvs=4#403a22e3d6e74102aebd69f83f5d721b](../Project%20Planning%20(quests)%203798a71e7c5d4a888cad9a7d25a1275c.md)
            
        - ✅ Fixed bug `ablated_model =...` as this doesn’t modify model, so model.reset_hooks() also does nothing. Chnage to `model=...`
        - `get_MLPs_actv_mean()` means error: `means = t.zeros(size=(n_layers, batch, seq_len, d_model), device=model.cfg.device)`, then when using `means[layer, template_group] = mlp_output_means_for_this_template`, this broadcasts the mean to every batch. But when have only one batch,
    - ✅ **ablate head 9.1 and mlp 9 and see if corr**
        
        This is necessary (AND) beacuse is seeing if components are essential (no backups)
        
        RESULT: this destroys the ability of ‘1 2 3 4’ to predict ‘5’, and it instead predicts ‘4’
        
    - ablate head 9.1 and mlp 9 then generate longer outputs for math-word-problem inputs
        - ISSUE: in curr code, when a model has a hook, can only take inputs of a certain size (same as means dataset), so cannot ‘add new token on then get next token’ when gen output longer than 1 next pos
            - [Try: ablation not dependent on means_dataset seqlen, but on curr input’s seqlen](https://colab.research.google.com/drive/1OVMkA1IZKZLmKq2paGRIXykzm5fxrGIx#scrollTo=ecUNF1uRd7hk&line=1&uniqifier=1)
                - Go back to [this issue](../Project%20Planning%20(quests)%203798a71e7c5d4a888cad9a7d25a1275c.md)
                - proposed soln
                    
                    Instead of making means shape be `n_layers, batch, seq_len, d_model`, using seq_len from means dataset, we should create a means for the specific current input. That means using a new means dataset based on the current input len. (Eg. if "1 2 3 4 5 6", make new means dataset that's len 6). This is needed since we need to get a means value for each pos of the input.
                    
                    Thus, define and pass in new dataset, and change this: `batch, seq_len = len(means_dataset), means_dataset.max_len`
                    
                    Then add a NEW HOOK using new dataset. So if generating, need to do this every loop
                    
                    Alt, use zero ablation
                    
        - [https://www.notion.so/wlg1/Project-Planning-3798a71e7c5d4a888cad9a7d25a1275c?pvs=4#dee8ff214d014d88b06898cc7f073014](../Project%20Planning%20(quests)%203798a71e7c5d4a888cad9a7d25a1275c.md)
    - ✅ even if not more than one step, we need to ablate components based on new input lengths. this is just needing new `means_dataset` inputs for each input
        - ✅ ISSUE:
            
            ![Untitled](Done%20b715c92198314529880806d9f206803d/Untitled%2010.png)
            
            - ✅ If we trace this back to how `means` is constructed, we see that its pos is determined by:
                
                `batch, seq_len = len(means_dataset), means_dataset.max_len`
                
                But `dataset_2.max_len = 12`, not 13. So it’s something else.
                
            - Place function in colab, and pdb in it (cannot do this if get code from repo)
                
                ipdb> means.shape
                torch.Size([12, 1, 12, 12, 64]) 
                
                `# n_layers, batch, seq_len, n_heads, d_head`
                
                ipdb> means[0].shape  `# same as hook.layer() = 0`
                torch.Size([1, 12, 12, 64])
                
                `z: Float[Tensor, "batch seq head d_head"],`
                
                “at dimension 1” means at seq. But it looks like for all layers, means dim 1 is 12. 
                
                <<<
                
                `print(hook.layer())
                print(z.shape)
                print(means[hook.layer()].shape)`
                
                0
                torch.Size([1, 13, 12, 64])
                torch.Size([1, 12, 12, 64])
                
                So it’s z that has 13, so chatgpt got this wrong by saying “a” is z. Not questioning this wrong assumption led to not finding the issue.
                
                SOLN: `tokens = tokens[:, 1:]`
                
    - ✅ get new dataset_2 to mean ablate. What did pos_dict do?
        
        pos_dict isn’t used to get means. `get_heads_actv_mean()` runs `means_dataset.toks.long()` as input to get a `means_cache` , which has activations for every layer
        
        `means_dataset.toks` is from: 
        
        ```
        self.toks = torch.Tensor(self.tokenizer(texts, padding=True).input_ids).type(
                    torch.int
                )
        ```
        
        Notice your dataset_2 DIDN’T corrupt the input “Today is…” but was the same. So make sure your corr prompts_list in its [’text’] key uses a corr vers with same len as clean input
        
        - ✅ [Manage to](https://colab.research.google.com/drive/1OVMkA1IZKZLmKq2paGRIXykzm5fxrGIx#scrollTo=ubcTjRDf6ETO&line=20&uniqifier=1) corrupt prompt using:
            
            ![Untitled](Done%20b715c92198314529880806d9f206803d/Untitled%2011.png)
            
            `tokens = tokens[:, 1:]`
            
            ![Untitled](Done%20b715c92198314529880806d9f206803d/Untitled%2012.png)
            
    - ✅ SOLN: how get rid of EOS token in fornt when tokenizing
        
        ```
        # this turns string into LIST OF TOKEN IDS
        tokens = model.to_tokens(reference_text).to(device)
        tokens = tokens[:, 1:] # get rid of prepend bos when using model.to_tokens
        
        # this turns it INTO LIST OF STRINGS WITH SPACE CHAR IN FRONT
        # each string in list correspond to tokens from token id list
        model.tokenizer.tokenize(text) # this doesn't use prepend bos
        ```
        
    - 🐣 [**Mean ablate for model generation**](https://colab.research.google.com/drive/1OVMkA1IZKZLmKq2paGRIXykzm5fxrGIx#scrollTo=SW9uBUML2gc4&line=1&uniqifier=1)
        
        Continues [this](../Project%20Planning%20(quests)%203798a71e7c5d4a888cad9a7d25a1275c.md)
        
- ✅ overleaf: update new sections and to-do plan

Steer numbers to ranks of single tokens

- 🐣 Steer numbers to ranks of single tokens using CAA code
    
    analogous_domain_steering_explora.ipynb
    
    - 🐣 check https://github.com/montemac/activation_additions : Uses TL
        - [scripts](https://github.com/montemac/activation_additions/tree/main/scripts)/**addition_clean_reimplementation.py:** [https://chatgpt.com/c/885b4a25-837d-487c-a703-5bd289d23d5a](https://chatgpt.com/c/885b4a25-837d-487c-a703-5bd289d23d5a)
        
        > For example, if we run `gpt2-small` on the prompt "I went to the store because", the residual streams line up as follows:…
        > 
    - ✅ get simplified code to work for 2 prompts for llama-2
    - reproduce past paper results for gpt-2 small and xl
- ✅ [ActivationAddition_explora.ipynb:](https://colab.research.google.com/drive/1MBqXSzO-wUqoAVevWRwqa0KWd_TPwD5h) Explore existing code
    
    Doesn’t work
    
    - first - January
    - ranks - months
- ✅ [Test llama-2: it can do “2 days after Tuesday is”](https://colab.research.google.com/drive/1E-T03wmDOtFiIN44xxqkeStyAncmQnwS#scrollTo=psa8bcKnGBE5&line=2&uniqifier=1)
    
    But it CAN’T do "Three days after Tuesday is”
    
    - 🐣 can it still do this if we corrupt the sequence?

Train SAEs on GPT2 Small [reproduce s.head feature steering]

- ✅ [GPT2_SAE_MLP0_seqcont.ipynb](../../Interpreting%20Steering%20Behaviors%20via%20Analogous%20Feat%208e01703d090b40ddbbb9ed25baec5b60/Project%20Planning%20b4b05f73d85e409f8409b209e44ed692.md)
    - ✅ Train it on integers, then see the highest features that light up on mod-10 tokens ending in 3
        
        NOTE: Most important means highest change in output after ablating. But here, we look for highest activations on these tokens. However, this doesn't mean much because certain features may fire highly for all numbers in general! So use the paper's definition of 'most important’
        
    - find most impt feature
        
        To replace feature in LLM
        
        - ✅ replace feature actvs from encoder output, then get decoder output
            - ✅ get feature neuron actvs and ablate a neuron to 0
            - ✅ reconstruct the actvs
                - ✅ see “GPT2small_SAE_train_demo_v1.ipynb” → reconstruction loss
                    - ✅ ISSUE:  Find an alt way to get and alter feature actvs than existing
                        
                        ```
                        output_tuple = autoencoder.forward(post_reshaped)
                        acts = output_tuple[3]
                        ```
                        
                        SOLN: The curernt way to alter uses the code above, but this outputs to the end- the third tuple is actvs, and fourth tuple is outputs. So instead of `autoencoder.forward`, multiply by decoder W and add decoder bias
                        
        - ✅ replace LLM actvs in that layer with decoder output
            - ✅ see: ARENA [1.2] Intro to Mech Interp
            - ✅ see: seqcont_circuits/iter_node_pruning/`mlp_ablation_fns.py` (mult)
            - ✅ NOTE: on using hooks
                
                ```
                # if you use run_with_cache, you need to add_hook before
                # if you use run_with_hooks, you dont need add_hook, just add it in fwd_hooks arg
                ```
                
            - ✅ h_reconstructed `torch.Size([20, 2, 3072])` needs to rearrange
                
                This is the output of two SAEs. We only need one, so `h_reconstructed[:, 0, :]`. Then, we can rearrange it to LLM dims (before, could not do this with two SAEs).
                
        - ✅ find change in output: look at logits after new orig fwd pass after ablating
            - see: ablate_seqcont_then_math__explora.ipynb
            - (to measure su head output) see: allTasks_nextScores.ipynb
        - 🐣 loop thru replacing each feature actvs with corrupted
        - ✅ [https://www.notion.so/wlg1/Repo-Structure-b81c8e1ce00b4bb48ea4e6f1a390c45d?pvs=4#7088e5c4d74f4efdbb521412b2caefb9](https://www.notion.so/Repo-Structure-b81c8e1ce00b4bb48ea4e6f1a390c45d?pvs=21)
        - ✅ Get output after successor heads, not after entire model
            
            NOTE: successor head can turn ‘3’ into ‘4’, but the model itself will turn ‘3’ into ‘.’ if there is no sequence in the prompt.
            
            - ✅ Get OV score code
            - ✅ ISSUE: OV score code uses "blocks.0.hook_resid_post", but SAE was trained on 'blocks.0.mlp.hook_post'
                - ✅ 'blocks.0.mlp.hook_post' vs  "blocks.0.hook_resid_post"
                    - **`blocks.0.mlp.hook_post`**: A hook placed after the MLP in the first transformer block, useful for inspecting/modifying the output of the MLP.
                    - **`blocks.0.hook_resid_post`**: A hook placed after the residual connection in the first transformer block, useful for inspecting/modifying the final output of the block before normalization.
                - ✅ SOLN: Which one of those outputs can be multiplied by OV matrix of an attention head (has the right dims)?
                    
                    Needs input size of : batch_size, seq_len, d_model
                    
                    Given this, **`blocks.0.hook_resid_post`**
                    
                
            - ✅ Re-train SAE on blocks.0.hook_resid_post activations
                
                Unablated: Return ‘4’ if input ‘3’
                
                Ablated: Return ‘third’ if input ‘3’ (ablate feature 0 of SAE)
                
                Interestingly, this “shifts” a number to a rank. Why?
                
            - ✅ save this SAE (much depends on randomness during training)
        - loop thru replacing each feature actvs with corrupted
- ✅ ISSUE: feature activations are all 0, and so decoder bias is broadcast (so the feature ablation of neuron idx 0 did nothing as everything in actvs was already 0). This makes the OV score be “third”
    
    sparsity means most activations should be 0
    
    find which feature neurons (rows) are not all 0s- ablate these
    
    if feature neurons are mostly 0, how can the reconstruction be similar to the original if it just broadcasts the bias? the original didn’t have so many repeats
    
    - perhaps SAE was not trained long enough. Train it on more steps
        - ✅ 5000 steps: same issue, but now broadcasted bias makes succesor head output be “ innumerable”
        - ✅ 10000 steps: sometimes is all 0, sometimes there are 5 VALUES out of 61435 (0.01%) that are non zero (these aren’t counting rows). This predicts ‘must’. **Reconstruction is bad-** because the ablation didn’t do much (first row likely all 0s anyways) it’s the reconstruction that messes up.
            - actually feature coeffs (encoder output actvs) shoudl be a 1D vector?
                - it is 1D, but acts is `[20, 2, 3840]` because of batch*seqlen and # SAE.
- ✅ rename above as “GPT2_SAE_MLP0_seqcont_explora_v1”
- 🐣 [GPT2_SAE_MLP0_seqcont_explora_v2](https://colab.research.google.com/drive/1fxFZynvhH0IkvE2WzUWUAqntQY4h7s5G).ipynb: clean up nb to only reproduce s.head’s “ablate impt feature, then pass thru s.head to see change in logits’
    - ✅ Use bigger SAE (bigger multp factor)
    - ✅ check reconstruction loss; how much reconstruction is similar to original actviation. see if gets same outputs in successor heads and model
    - try to lower reconstruction loss with better training params
        - (Cunningham et al) used small mult factor, R=2, and only 10 epochs?
        - s.heads: Training a sparse auto-encoder withD features and regularization coefficient λ... We used the hyperparameters D = 512 and λ = 0.3, with a batch size of 64, and trained for 100 epochs
- ✅ re-plan emnlp on overleaf + google docs
    
    i just reread the successor head paper and realized they already did do 'transformative mapping between analogous domains'. but they didnt do much with it. i think there's other approaches to try. i guess it's not about knowing the problem to solve, but about the solution that's come up. so just bc multiple rsch groups indp find the same problem doesnt mean anything, what matters is their solution
    
- ✅ explain succssor heads commutative diagram
    
    [https://chatgpt.com/c/e3511349-5a19-4862-8f0b-18ba33dbe477](https://chatgpt.com/c/e3511349-5a19-4862-8f0b-18ba33dbe477)
    
    Ensure that these operations commute, meaning the different paths yield the same results, confirming that the projections πN and πD are correctly learned and that the representations are consistent.
    
    ![Untitled](Done%20b715c92198314529880806d9f206803d/Untitled%2013.png)
    

Test prompts

- ✅ Spanish
    
    numseq_prompts, small, pt2.ipynb
    
    small cannot do spanish
    
    But llama-2 CAN do spanish. cannot do german
    
    - usually, BERT models do this (seee “Do Llamas work in English”) but GPT-2 has french
        
        For smaller models, e.g., GPT-2 (Radford et al.,
        2019) and Pythia (Biderman et al., 2023), MI approaches
        such as sparse probing (Gurnee et al.,
        2023) have revealed monosemantic French (Gurnee
        et al., 2023) and German (Quirke et al., 2023) language
        neurons and context-dependent German ngram
        circuits (subnetworks for boosting the probability
        of German n-grams when the monosemantic
        German context neuron is active) (Quirke et al.,
        2023).
        
- ✅ Llama-2 can do interval reasoning
    
    in two days it will be…
    
- ⚠️ Diverging sequence detection
    
    Two inputs sound similar, but diverge at a certain point
    
    unfortunately, llama-2 cant do multiplicative
    
- ✅ llama-2 may be able to do less than
    
    Bob is older than Steve. Bob is 10 and Steve is
    
    Bob is one year older than Steve. Bob is 6 and Steve is
    
    ISSUE: fails on this: Bob is three years older than Steve. If Bob is 6, how old is Steve?
    
- ✅ don’t rely on ‘utils’, use ‘generate next char’ code directly!
    
    utils test prompt is unreliable bc its answers rank changes based on what’s in example_answer!
    
    - prompt_llama2-7b_TLgen.ipynb
        - this works:
            
            ```
            prompt = "Bob is three years older than Steve. If Bob is 6, how old is Steve?"
            output = tl_model.generate(prompt, max_new_tokens=50, **generate_kwargs)
            print(output)
            ```
            
            Bob is three years older than Steve. If Bob is 6, how old is Steve?
            
            Answer: If Bob is 6 years old and Bob is 3 years older than Steve, then Steve is 6 - 3 = 3 years old.</s>
            
- ✅ [List of math reason prompts](../List%20of%20math%20reason%20prompts%207fad070b1fa5480e858b6fbef3217c2a.md)

Find Llama-2 circuits for additive vs mult vs fibo seqs

- ✅ run “2 4 6” - all single tokens
    
    Llama2_2468.ipynb
    
- ✅ run “1 2 3”- all single tokens
    
    Llama2_1234.ipynb
    
    - ✅ issue with Llama2_num_runModel_explrTests_v6.ipynb: the prompts go from (1,6), so last samples is “6 7 8 9” and supposed to predict ‘10’ as corr tok but you will get ‘1’. This means not accurate when getting logit diff score. The pos_dict used in dataset is also not matching the seq_pos_to_keep (does it matter?)
    - ✅ Llama2_num_runModel_explrTests_v4.ipynb: this is single samp 12345, but fails
        - useful code
            
            ```jsx
            # logits[:, -1, :] selects the last token for all batches, here batch size is 1
            last_token_logits = logits_original[:, -1, :]
            values, indices = torch.topk(last_token_logits, 5, dim = -1)
            for token_id in indices[0]:
                print(model.tokenizer.decode(token_id.item()))
            ```
            
- ✅ [Successor Heads Notes](../Successor%20Heads%20Notes%203d2a9c73e41d4ca7903790b6e7914124.md) : questions on proj mat training
- ✅ Llama2_2468: on impt ones, logit lens + ov scores
    - adapt `get_next_scores` for llama-2 (TL)
        
        llama2_plus2_nextScores.ipynb
        
        [https://colab.research.google.com/drive/11mvy5ZZud0Pyr6XYuoPBGpkfo9Hmp4tS](https://colab.research.google.com/drive/11mvy5ZZud0Pyr6XYuoPBGpkfo9Hmp4tS#scrollTo=sUnSHvA-Myx8)
        
        - ✅ chnage to attn.hook_z from attn.ln1
        - ✅ get rid of corr and incorr tokens, get rid of S pos for spaces
        - ✅ loop thru all heads for copy and next scores
        - ✅ next next score (+2)
            
            `next_word = str(int(prompt[word]) + 2)`
            
        - replace `cache_some` with `run_with_hook`
    - ✅ MLP logit lens
        
        [lllama2_plus2_logit_lens.ipynb](https://colab.research.google.com/drive/11mvy5ZZud0Pyr6XYuoPBGpkfo9Hmp4tS)
        
        The right or close answer happens at layer 17 to 18
        
        For +2, this gets the “number” around layer 17, but only gets to the right +2 number around layer 24. For plus 3, it’s around layer 20 or 25.
        
        It sems a SPECIFIC layer is not responsible for this correction. Instead, it’s general “layers around 20 to mid 20s” that does this. Why?
        
        - 1 2 3: immediately goes to 4
            
            17 ['isti', 'кт', 'udio', ' estaven', ' jack']
            18 ['4', '3', ' forward', 'Enable', 'кт']
            19 ['4', '3', ' inside', ' forward', '5']
            20 ['4', ' fourth', ' four', '3', '5']
            21 ['4', ' four', ' fourth', '3', ' cuatro']
            22 ['4', ' four', ' fourth', ' forward', ' cuatro']
            23 ['4', ' four', ' fourth', ' cuatro', '3']
            24 ['4', ' four', ' fourth', ' Four', ' cuatro']
            25 ['4', ' four', ' fourth', ' Four', ' cuatro']
            26 ['4', ' four', ' fourth', ' Four', ' vier']
            27 ['4', ' four', ' fourth', ' vier', ' cuatro']
            28 ['4', ' four', ' fourth', ' forth', ' vier']
            29 ['4', ' four', ' forth', ' fourth', ' vier']
            30 ['4', ' four', '<0x0A>', ' (', ' fourth']
            31 ['4', '5', '1', ' four', '0']
            
        - 2 4 6: wrestles with predicting 7 or 8
            
            17 ['xt', 'chor', 'wich', 'end', '7']
            
            18 ['7', 'chor', ' Dic', '8', 'amba']
            19 ['8', '7', '2', ' Dic', '3']
            20 ['7', '8', '2', '6', '1']
            21 ['7', '8', ' VII', '2', 'penas']
            22 ['7', '8', '9', '2', 'rugu']
            23 ['7', '8', 'veis', 'richt', ' ten']
            24 ['8', '7', ' eight', ' ten', '6']
            25 ['8', '7', ' eight', '<0x0A>', ' and']
            26 ['8', '7', ' eight', ' ten', ' and']
            27 ['8', ' eight', '<0x0A>', ' and', '7']
            28 ['8', '<0x0A>', ' in', ' and', ' (']
            29 ['8', '<0x0A>', ' eight', ' in', '1']
            30 ['8', '<0x0A>', ' (', '1', '2']
            31 ['8', '1', '2', '4', '0']
            
        - 0 2 4: similar
            
            17 ['chor', '0', '4', '2', '5']
            18 ['0', 'chor', '5', '2', '4']
            19 ['4', '3', '0', '5', 'chor']
            20 ['5', '6', '7', '8', '0']
            21 ['5', '6', 'olis', '7', '8']
            22 ['5', '6', '7', 'olis', '8']
            23 ['5', 'olis', '7', '6', 'Ő']
            24 ['6', '8', '5', 'olis', ' six']
            25 ['6', '8', ' six', 'olis', ' in']
            26 ['6', '8', ' six', '7', ' in']
            27 ['6', '8', ' in', '7', '<0x0A>']
            28 ['6', '8', ' in', '7', '<0x0A>']
            29 ['6', ' in', '<0x0A>', '8', '7']
            30 ['6', '<0x0A>', '8', ' in', ' (']
            31 ['6', '8', '0', '1', '7']
            
        - 2 4 6 8: similar
            
            17 ['xt', '8', 'chor', 'conde', 'end']
            18 ['8', 'xt', ' "_', 'penas', 'amba']
            19 ['8', ' charm', '9', ' Um', '7']
            20 ['9', '8', 'penas', 'chor', ' Chor']
            21 ['9', 'penas', '8', '<0xA7>', 'ъ']
            22 ['9', 'penas', '8', ' ten', ' Ten']
            23 ['9', 'penas', 'üb', '8', ' ten']
            24 ['9', '8', ' ten', ' Ten', ' rör']
            25 [' ten', '8', ' Ten', 'penas', '9']
            26 [' ten', '8', ' Ten', '9', 'penas']
            27 [' ten', '9', '8', ' Ten', '<0x0A>']
            28 ['9', ' ten', '<0x0A>', ' in', '1']
            29 [' ten', '9', '<0x0A>', '1', ' Ten']
            30 ['<0x0A>', '1', ' (', '9', ' ten']
            31 ['1', '9', '2', '0', '4']
            
        - 0 3 6
            
            16 ['Ő', ' fill', ' pla', '7', ' Lad']
            17 ['0', '6', ' fill', '3', '7']
            18 ['0', '7', '6', '4', '5']
            19 ['7', '8', '6', '3', '0']
            20 ['7', '8', '6', '0', '1']
            21 ['7', '8', '1', '6', '0']
            22 ['7', '8', '9', '1', '0']
            23 ['7', '8', '9', '6', 'aucoup']
            24 ['7', '9', '8', '6', '0']
            25 ['9', '7', 'aucoup', '8', 'aden']
            26 ['9', '8', '7', ' in', '1']
            27 ['9', '8', '7', ' in', ' (']
            28 ['9', '8', ' (', ' in', '7']
            29 ['9', '8', ' in', '<0x0A>', '1']
            30 ['9', '8', '1', '<0x0A>', ' (']
            31 ['9', '8', '1', '7', '0']
            
        - 3 6 9
            
            15 ['Ő', ' fill', 'dex', '9', ' Lad']
            16 ['9', '7', 'Ő', '8', ' fill']
            17 ['9', '7', 'Ő', '8', ' refer']
            18 ['9', '7', '8', '4', '1']
            19 ['9', '7', '8', '1', ' numer']
            20 ['9', '1', '7', '8', ' numer']
            21 ['1', '9', '7', ' numer', ' Ten']
            22 ['1', '9', '7', ' numer', ' sus']
            23 ['1', '9', ' straight', ' numer', '7']
            24 ['1', '9', ' ace', ' aj', ' straight']
            25 ['1', ' aj', ' push', ' ace', 'chor']
            26 ['1', ' push', '2', ' ace', 'chk']
            27 ['1', '2', ' push', '8', '5']
            28 ['1', '2', '5', '8', ' push']
            29 ['1', '2', '5', '4', '7']
            30 ['1', '5', '2', '<0x0A>', ' (']
            31 ['1', '5', '2', '4', '3']
            
        
        Also notice that “cuatro” and “vier” are one of the top 5 tokens in “1 2 3”. So spanish nums are closely tied to english nums, similar to months and numbers.
        
- ✅ write up prelim findings on overleaf (helps direction)
- ✅ run node ablation on “uno dos tres cuatro”
    
    [Llama2_spanish.ipynb](https://colab.research.google.com/drive/1QUcybaXsIJrSvL77eph9ejramiUM-z5d#scrollTo=4dlDnLnesbfW)
    
    Unlike “2 4 6 “, these number words should not put a space at the end of the prompt, else they get the wrong answer. This is because the number words in Spanish ALREADY have the space in front. 
    
- ✅ why gen results diff from ‘prompt test’ and ‘ablation’ nb?
    - NO- issue with model.generate: it does random sampling, so is not always the top picked answer
        1. **generate_kwargs Explanation**:
            - `do_sample = False`: This ensures deterministic output, meaning that the model will not use random sampling but will instead choose the token with the highest probability at each step. This is important for reproducibility and comparison.
            - `top_p = 1.0`: This parameter is related to nucleus sampling (top-p sampling). Setting it to 1.0 effectively disables nucleus sampling, meaning that it won't limit the pool of tokens to a top percentage of the cumulative probability distribution.
            - `temperature = 1.0`: The temperature parameter controls the randomness of predictions by scaling the logits before applying softmax. A temperature of 1.0 means no scaling, thus maintaining the original probability distribution.
        2. **Issue with `model.generate`**:
            - The `model.generate` function indeed defaults to random sampling (if `do_sample` is set to `True`), which can lead to non-deterministic outputs. This means that even if you use the same prompt, you might get different continuations each time you generate text.
            - To ensure deterministic behavior, you should set `do_sample` to `False`, as mentioned in the kwargs. This way, the model will always pick the token with the highest probability at each step, leading to consistent outputs.
    
    SOLN :
    
    albation nb used: `tokenizer = LlamaTokenizer.from_pretrained(LLAMA_2_7B_CHAT_PATH, use_fast= False, add_prefix_space= False)`
    
- ✅ org findings into expm pages. org brainstorm into pages
    - ✅ [EMNLP overall plan](https://www.notion.so/EMNLP-overall-plan-30c3e0ec30454843918ffc841d2422f6?pvs=21)
- ✅ re-run foreign lang ablation
    
    But is “no space at end of prompt” because the tokenizer is different? No- in both cases of the tokenizer, this is the case. The only difference is that “uno dos tres” now completes correctly, instead of needing cuatro.
    
- 🐣 try kojima code
    - [https://github.com/kojima-takeshi188/lang_neuron/blob/main/scripts/generate_seq_lang.py](https://github.com/kojima-takeshi188/lang_neuron/blob/main/scripts/generate_seq_lang.py)
        - [https://chatgpt.com/c/daf7d976-004c-48de-b529-2f71eed1e016](https://chatgpt.com/c/daf7d976-004c-48de-b529-2f71eed1e016)
            
            The `force_units_hooks` function is used to manipulate specific neurons within the model, setting their activation to the specified forcing values.
            
    - [https://github.com/kojima-takeshi188/lang_neuron/blob/main/selfcond/generation.py](https://github.com/kojima-takeshi188/lang_neuron/blob/main/selfcond/generation.py)

Ablation hook for math reasoning word problems

- ✅ perform mean hook ablation for multiple tokens
    
    ablate_seqcont_then_math__explora.ipynb, section “Mean ablate for model generation”
    
    [https://colab.research.google.com/drive/1OVMkA1IZKZLmKq2paGRIXykzm5fxrGIx#scrollTo=SW9uBUML2gc4](https://colab.research.google.com/drive/1OVMkA1IZKZLmKq2paGRIXykzm5fxrGIx#scrollTo=SW9uBUML2gc4)
    
    - debug why at i==3 (the third new gen token) it runs into `The size of tensor a (15) must match the size of tensor b (16) at non-singleton dimension 1`
        
        - For some reason, the model must accept tokens of size `[1,15]`, meaning the pos_dict put in dataset_2 only contains 15 positions. 
        
        - SOLN: We see that this is the fault of the tokenizer!
            
            ipdb> corr_text
            'What comes after X is Y, and two days after is Z?\n\n'
            
            ipdb> model.tokenizer.tokenize(corr_text)
            ['What', 'Ġcomes', 'Ġafter', 'ĠX', 'Ġis', 'ĠY', ',', 'Ġand', 'Ġtwo', 'Ġdays', 'Ġafter', 'Ġis', 'ĠZ', '?', 'ĊĊ']
            ipdb> model.tokenizer.tokenize('What comes after Monday is Tuesday, and two days after is Z?\n')
            ['What', 'Ġcomes', 'Ġafter', 'ĠMonday', 'Ġis', 'ĠTuesday', ',', 'Ġand', 'Ġtwo', 'Ġdays', 'Ġafter', 'Ġis', 'ĠZ', '?', 'Ċ']
            
    - `model.to_tokens and model.tokenizer.tokenize` have different lengths?? No; it’s this code’s fault: `tokens = t.cat([tokens, next_token[None, None]], dim=-1)`
        
        We should construct the tokens from the NEW sequence, not just add it on!
        
        ```jsx
        reference_text = reference_text + next_char
        tokens = model.to_tokens(reference_text).to(device)
        tokens = tokens[:, 1:]
        ```
        
        For some reason, we don’t need `tokens = tokens[:, 1:]` for the corrupted text? This is because we use two different ways to get clean and corr; for corr, we use:
        
        ```jsx
        self.toks = torch.Tensor(self.tokenizer(texts, padding=True).input_ids).type(
                    torch.int
                )
          ...
        model.run_with_cache(
                means_dataset.toks.long(),
        ```
        
    - Summary of issue-soln
        
        We cannot just do `tokens = [t.cat](http://t.cat)` because the new corrupted string that adds on the previous token, when tokenized, can have different number of tokens that the clean tokens (`tokens)`
        
- ✅ measure the correct answer for multiple tokens
    
    [**logit diff for mult tok answers**](https://colab.research.google.com/drive/1OVMkA1IZKZLmKq2paGRIXykzm5fxrGIx#scrollTo=jtaV1q3SBHow&line=1&uniqifier=1)
    
    1. first, we get how long the correct string answer is in terms of token IDs
        1. we use only 1 sequence first, so this is easy
    2. then, for model after hook: 
        1. we keep on passing in the output autoregressively for the number of tokens the correct answer is. each pass, we reset and add in a NEW hook based on a NEW clean dataset (that adds a new char on) and a NEW corrupted dataset
        2. each pass, we measure the logit of the correct token of that pass, taking the difference with the logit of the last sequence member
    3. we add up the logit diffs as an ablation score
- ✅ seqcont + math reasoning ablation experiments using just sub-circ
    - [pure seq 1 2 3](https://colab.research.google.com/drive/1OVMkA1IZKZLmKq2paGRIXykzm5fxrGIx#scrollTo=H5H-d2URUSVJ&line=1&uniqifier=1)
    - [seqcont word problems](https://colab.research.google.com/drive/1OVMkA1IZKZLmKq2paGRIXykzm5fxrGIx#scrollTo=aB5OqTVcYigX&line=1&uniqifier=1)
    - 2 4 6 (GPT-2 CAN’T do this even clean, so try it on llama-2)
- ✅ [ablate_seqcont_thenMath_explora_v2](https://colab.research.google.com/drive/1wf9Xsd4Qbq7rp-v4DS8_ozlMb_3sdRxI?authuser=4): clean up and turn redundant code into fns
    
    [ablate_seqcont_thenMath_explora](../Expm%20Results-%20EMNLP%20df8b503260cf48d987a58d25103b6638/ablate_seqcont_thenMath_explora%202dd1d8fc1ea6468084bf1811562b836f.md) 
    
    - ✅ turn redundant code into fns
    - ✅ try diff circs on more tasks for gpt2-small
- ✅ test llama-2 gen prompts- clean and corrupt
    
    [llama2_ablate_prompts_explora_v1.ipynb](../Expm%20Results-%20EMNLP%20df8b503260cf48d987a58d25103b6638/llama2_ablate_prompts_explora_v1%20ipynb%20b1754a3f537e459f8cda41e71979bd75.md) 
    
    - ✅ how come i get different answers for:
    model.generate(prompt, max_new_tokens=1, **generate_kwargs)
        
        VS
        
        logits = model(tokens)
        next_token = logits[0, -1].argmax(dim=-1) # Get the predicted token at the end of our sequence
        next_char = model.to_string(next_token)
        
        using same tokens?
        
        SOLN: You MUST use `tokenizer = LlamaTokenizer.from_pretrained(LLAMA_2_7B_CHAT_PATH)`
        
        not `tokenizer = LlamaTokenizer.from_pretrained(LLAMA_2_7B_CHAT_PATH, use_fast= False, add_prefix_space= False)` [used in llama_1234, and files before may 26th]
        
        For llama-2 (unlike gpt2), it seems both cases DON’T use:  `tokens = tokens[:, 1:] # get rid of prepend bos when using model.to_tokens`
        
    - ✅⚠️ ISSUE: “uno dos tres” still doesn’t work, but “uno dos tres cuatro” does work
        
        SOLN: comment out `tokens = tokens[:, 1:]` in clean_gen()
        
    - ✅⚠️ [https://stackoverflow.com/questions/68166721/pytorch-fails-with-cuda-error-device-side-assert-triggered-on-colab](https://stackoverflow.com/questions/68166721/pytorch-fails-with-cuda-error-device-side-assert-triggered-on-colab)
        
        Most of the time CUDA Runtime Errors can be the cause of some index mismatching so like you tried to train a network with 10 output nodes on a dataset with 15 labels. And the thing with this CUDA error is once you get this error once, you will recieve it for every operation you do with torch.tensors. **This forces you to restart your notebook.**
        
    - ✅ ISSUE: if add llama2’s space to string, it’s an empty string, so won’t increment anything.
        
        ```
          clean_text = clean_text + next_char
          tokens = model.to_tokens(clean_text).to(device)
        ```
        
        In constrast, `tokens = torch.cat([tokens, next_token[None, None]], dim=-1)` WILL add it. Just say if next_char is ‘’, make it a space instead.
        
    - ✅ debug why ablating nothing gives wrong answer for 1 2 3
        - ✅ what is token id 29871 in llama-2? is it empty space?
            
            [https://chatgpt.com/c/ed75af00-9cf1-4005-aed3-f7e6967a7063](https://chatgpt.com/c/ed75af00-9cf1-4005-aed3-f7e6967a7063)
            
            - is this a valid way to handle that token?
                
                The approach of appending the next character to the cleaned text and then tokenizing the resulting string can be valid, but there are some considerations to ensure its effectiveness and efficiency. Here's a step-by-step breakdown:
                
                ### Code Review:
                
                ```python
                clean_text = clean_text + next_char
                tokens = model.to_tokens(clean_text).to(device)
                
                ```
                
                ### Considerations:
                
                1. **Incremental Tokenization**:
                    - Continuously appending characters and re-tokenizing the entire string can be computationally expensive, especially for longer texts. Incremental tokenization techniques can help optimize this by only tokenizing the new character(s) and updating the existing token list accordingly.
                2. **Handling Special Tokens**:
                    - Ensure that special tokens (like `</s>`, `<s>`, etc.) are correctly managed in your tokenization process. Mismanagement can lead to incorrect representations of the text.
                3. **Device Management**:
                    - Moving tokens to the appropriate device (e.g., GPU) after tokenization is crucial for model inference. However, ensure that this operation is efficient and does not cause bottlenecks.
                
                ### Example Implementation:
                
                Here’s an enhanced approach considering incremental tokenization and efficient device management:
                
                ```python
                # Initialize cleaned text and tokens
                clean_text = ""
                tokens = []
                
                def add_char_and_tokenize(clean_text, next_char, model, device):
                    # Append the next character to the text
                    clean_text += next_char
                
                    # Tokenize only the new character
                    new_tokens = model.to_tokens(next_char).to(device)
                
                    # Append new tokens to the existing token list
                    tokens.extend(new_tokens)
                
                    return clean_text, tokens
                
                # Usage example
                clean_text, tokens = add_char_and_tokenize(clean_text, next_char, model, device)
                
                ```
                
                ### Benefits of the Improved Approach:
                
                1. **Efficiency**: Only the new character is tokenized, reducing redundant computations.
                2. **Scalability**: Better performance with longer texts as the whole text isn't re-tokenized each time.
                3. **Maintainability**: Clearer separation of concerns, making the code easier to maintain and debug.
                
                ### Conclusion:
                
                While the initial approach is functional, optimizing it by tokenizing only new characters and efficiently managing device transfers can significantly improve performance and scalability. This is especially important in real-time applications or when working with large datasets.
                
        - ✅ try this in generate prompt lsits longer:
            
            ```
                # tokens_as_strs = model.tokenizer.tokenize(text)
                tokens_as_strs = model.to_string(tokens)[0].split()
            ```
            
            RESULT: no diff
            
        - ✅ try new way to concat corr tokens
            
            ```
            corr_text = model.to_string(corr_tokens)[0]
            corr_tokens = torch.cat([corr_tokens, next_token[None, None]], dim=-1)
            ```
            
            RESULT: no diff and gets into an error after some iters
            
        - ✅ Try using llama2_1234’s way of creating prompt_dict and pos_dict in dataset (there may be a tokenizer and to_token misalignment)
            
            RESULT: no diff
            
        - ✅ Try modify circuit and seq pos in hook_fn to use that of llama2_1234
            
            RESULT: no diff
            
        - ✅ setting next_char as `▁`
            
            RESULT: no diff
            
        - ✅ IMPT: We find there is a mistranslation back and forth for the space char
            
            ![Untitled](Done%20b715c92198314529880806d9f206803d/Untitled%2014.png)
            
            We shouldn’t be using `model.to_string`  on tokens either to get the string of this token, as this translates it into  <s>, which is also on the token ID “1”, too like ‘’
            
            ![Untitled](Done%20b715c92198314529880806d9f206803d/Untitled%2015.png)
            
        - ✅ TRY: a combo of + for text (avoid `to_string`) but token concat for tokens (this preserves tokens, avoiding needing it to translate it back and forth from str using `to_token`)
            
            Also, for + using text, make next_char be ‘ ‘, but keep the tokID as 29871 instead of 1
            
            RESULT: no diff
            
        - ✅ Look at `ablate_head_MLP` again. Its positions are SPECIFIC to the fixed FIRST STRING, whereas `ablate_head` adapts to the longer strings! So it’s the hook_fn that likely has an issue!
            
            No; actually it does adapt
            
        - Look at Dataset() and the ablation funcs again to see how they use tokenize, encode, etc. Make sure not to mix these up.
            - you DON’T want to create a new text that uses <s> in front! So avoid using `to_string(tokens)`
            - `mask_circ_heads()` depends on `word_idx`. word_idx is a DICTIONARY such that its keys are S1, etc position keys from `SEQ_TO_KEEP`, and the values are whose values are a tensor with an element for each prompt. The element is the targ token's ind at that prompt. It is built by `pos_dict` , so it’s highly important that its keys correspond to the actual tokens
            - In `get_heads_actv_mean()`, they use `model(means_dataset.toks)`, which is obtained by : `self.toks = torch.Tensor(self.tokenizer(texts, padding=True).input_ids)        )`
                
                This is okay, because tokens ALSO has a <pad> in front. The original tokens (before adding things on) has this too.
                
            - paste `mask_circ_heads(`) in colab as a function to override the imported one, and pdb to see what is being masked (kept using 1 in mask). EVERYTHING should be kept.
                
                However, we find that the last position isn’t kept! For each layer, for each head, we see this for the last position: `heads_and_posns_to_keep[0][:, -1, :]` is all false. 
                
                The seq pos which are set to 1 are dictated by this:  `indices = means_dataset.word_idx[seq_pos]`. The `word_idx` is dictated by `pos_dict`
                
                heads_and_posns_to_keep[0].shape
                torch.Size([1, 7, 32])
                
                indices
                tensor([5])
                
                circuit.keys()
                dict_keys(['S0', 'S1', 'S2', 'S3', 'S4', 'S5'])
                
                Indices loops through every pos in circuit.keys().  This has 6 positions. But mask has 7 positions. So one position is missing in `circuit`!
                
                The shape of `mask` comes from `means_dataset.max_len`. That’s why it has 7. 
                
                ```
                self.max_len = max(
                            [
                                len(self.tokenizer(prompt["text"]).input_ids)
                                for prompt in self.prompts
                            ]
                        )
                ```
                
                means_dataset.toks.shape
                torch.Size([1, 7])
                
                So it SHOULD have 7. But circuit has 6. Instead of looping through `for i in range(len(model.tokenizer.tokenize(means_dataset.prompts[0]['text']))):`, it should be like max_len and use the `input_ids`
                
                SOLN:
                
                means_dataset.prompts[0]['text']
                '5 3 9'
                
                model.tokenizer(means_dataset.prompts[0]['text']).input_ids
                [1, 29871, 29945, 29871, 29941, 29871, 29929]
                
                The culprit is the <PAD> at the start; because we pad its position, we don’t pad the last position!
                
                Not only do we need to update circuit and the pos to keep, we need to update pos_dict, which matches the type of pos (S1; in name movers, this can be Subject, which is not always at “1” but varies) to where that type is in the sequence
                
            - There is still an issue. If we remove the mlp hook, there is still an issue. IF we rmv the head hook, it’s fixed. So the issue is at the head hook.
                
                heads_and_posns_to_keep[0].unsqueeze(-1).shape
                
                torch.Size([1, 7, 32, 1])
                
                z = torch.empty([1, 7, 32, 2])
                
                t.where(heads_and_posns_to_keep[0].unsqueeze(-1), z, 2)
                
                this replaces nothing because all is masked (bool all true). note it broadcasts
                
                in `add_ablation_hook_MLP_head()`
                
                ```
                    if all_entries_true(components_to_keep) == False:
                        pdb.set_trace()
                ```
                
                components_to_keep[0].shape
                torch.Size([1, 8, 32])
                
                So, initially it is good. But when it appends a token to the end, this is not updated and the last position is still all False.
                
                This is because Dataset(prompts_list_2, pos_dict, model.tokenizer) updates from prompts_list_2, but not tokens! 
                
                No; it’s because our function was importing other functions that overwrote our new ones in its block, AND it was using dataset_2 instead of mean_dataset, AND it didn’t do num_pos like in the generation loop
                
                SOLN: Get rid of end pos in both circuit, seqpostokeep!
                
    - ✅ debug: corrupted, when it grows, is tokenized differently than clean. so you have different positions.
        - error
            
            ![Untitled](Done%20b715c92198314529880806d9f206803d/Untitled%2016.png)
            
        
        Just try 0 ablation. Instead of obtaining means from corr, just set clean’s actvs to 0
        
    - ✅ ablate only previous + zero ablation reasoning
        
        (Put details of ablation in appendix)
        
        OR- there is no need for further ablation. If the new tokens are just the same for both clean and corr, we can just ablate the previous tokens (though this may change too with tokenizer, so perhaps do 0 ablation) and generate the rest without any new hooks. This is because the sequential information is already lost for those specific tokens before. 
        
        The reason we do 0 is because dos may be processed as (d, os) or (dos) depending on the entire input. So we can’t find the mean repalcement of it as there’s no actual 1-1 token correspondence. Thus, when processing several prompts, 0 is more reliable. (Zero and mean ablation have similar results, though mean should be more accurate.) Many papers use zero ablation.
        
        By destroying how components X processes information at those positions/tokens P, when the attention pattern QK is calculated, that is enough to not have information on what should be next if the circuit relies on those components. BUT if other unablated components (as backup) can also process P, then the correct generation can still work. We want to show those components X are necessary- there are no backups. Or, we include the backups in set X. 
        
        If we just use this circuit on more complicated prompts, it may not work as they need more components. But showing those components are essential so that knocking out just a few is enough to disrupt that function 
        
        Zero is harsher, more extreme differences. Mean allows more precise showing of not just different number but different specific number, as it'd still generate a number, whereas zero likely won't. 
        
        Actually if we just append a token we can still use mean. But it's not as accurate as if we tokenize the entire sentence, as that's what the model was trained on. Or is it?
        
- ✅ In a transformer, when the last output is put back to the input, is it concatenating tokens, or adding to the string then tokenizing the new string? What is the new tokenized string tokenizes the words in a new way? For instance, before it could use "dos" as one token, but now it uses dos as two tokens that correspond to "d" and "os”
    
    When the last output of a transformer is fed back into the input, it typically involves taking the generated text and tokenizing it again before re-feeding it into the model. This process usually works as follows:
    
    1. **Output Generation:** The model generates a sequence of tokens, which are then decoded into a string of text.
    2. **Text Concatenation:** The generated string is concatenated to the existing input text.
    3. **Re-Tokenization:** The concatenated text is then tokenized again to produce a new sequence of tokens.
    
    The tokenization process is crucial because the way text is split into tokens can affect how the model interprets and processes the input. Tokenization is often done using a predefined tokenizer that splits the text into units (tokens) based on the vocabulary the model was trained on.
    
    ### Tokenization Variability
    
    When the new concatenated string is tokenized, it may indeed be tokenized differently than the original input. For example:
    
    - **Initial Input:** "The quick brown fox jumps over the lazy dog."
    - **Generated Output:** " And runs away."
    - **Concatenated String:** "The quick brown fox jumps over the lazy dog. And runs away."
    
    The tokenizer will then process this concatenated string. If the tokenizer splits "runs" into "r" and "uns" instead of a single token "runs", this could affect how the model interprets the new input.
    
    ### Handling Tokenization Differences
    
    Transformers typically use tokenization schemes like Byte-Pair Encoding (BPE) or SentencePiece, which are designed to handle subword units efficiently. However, these schemes can result in different tokenization for the same word in different contexts.
    
    For instance:
    
    - **Initial Tokenization:** "dos" might be a single token.
    - **Re-Tokenization:** "dos" might split into "d" and "os" if it appears in a different context.
    
    The model is trained to handle such variations, but it's important to be aware that re-tokenization can lead to differences in how the input is processed. This variability is part of what makes transformer models robust yet sometimes unpredictable in their tokenization behavior.
    
    ### Practical Implications
    
    In practice, you might not have direct control over how tokenization changes unless you modify the tokenizer or use a custom tokenization scheme. Understanding how your tokenizer works and being aware of these potential changes is important for interpreting the model's behavior when feeding outputs back into inputs.
    
- ✅ is this for gpt2 and llama2?
    
    Yes, this process of output generation, text concatenation, and re-tokenization applies to both GPT-2 and LLaMA2, as well as other transformer-based models. Here’s a more detailed look at how it works for these specific models:
    
    ### GPT-2
    
    1. **Output Generation:** GPT-2 generates a sequence of tokens based on the given input. These tokens are then decoded into a string of text.
    2. **Text Concatenation:** The generated text is concatenated to the original input text.
    3. **Re-Tokenization:** The concatenated text is re-tokenized using GPT-2's tokenizer, which employs Byte-Pair Encoding (BPE). BPE can result in different tokenizations depending on the context and preceding text.
    
    For example:
    
    - **Initial Input:** "The dog"
    - **Generated Output:** " jumps over"
    - **Concatenated String:** "The dog jumps over"
    - **Re-Tokenization:** The new string might be tokenized differently than if "jumps over" was tokenized in isolation.
    
    ### LLaMA2
    
    1. **Output Generation:** LLaMA2 generates tokens based on the input sequence, which are then decoded into text.
    2. **Text Concatenation:** The generated text is concatenated with the original input text.
    3. **Re-Tokenization:** The concatenated text is tokenized again using the tokenizer designed for LLaMA2. Like GPT-2, LLaMA2 uses subword tokenization techniques that may split words differently depending on their context.
    
    For example:
    
    - **Initial Input:** "el gato"
    - **Generated Output:** " está en"
    - **Concatenated String:** "el gato está en"
    - **Re-Tokenization:** Depending on the context, "está en" might be tokenized as distinct tokens, potentially leading to different token splits.
    
    ### Tokenization Differences
    
    Both GPT-2 and LLaMA2 use tokenization schemes that are sensitive to the context and preceding text. This means that the same word or phrase might be tokenized differently when it appears in a different context within the input sequence. This is an inherent feature of how subword tokenization methods like BPE or other variants work.
    
    - **Byte-Pair Encoding (BPE):** Used by GPT-2, where common subword units are merged into single tokens based on their frequency.
    - **SentencePiece or Custom BPE:** Used by LLaMA2, which also employs subword tokenization but might have different rules or vocabulary.
    
    ### Practical Implications
    
    When using these models, it’s important to be aware that:
    
    - **Tokenization Consistency:** The way text is tokenized can change based on the context, affecting model behavior.
    - **Handling Variability:** While models are generally robust to these variations, understanding tokenization can help in debugging and improving input preprocessing.
    
    In summary, both GPT-2 and LLaMA2 handle output feedback through text concatenation and re-tokenization, and they may tokenize the same string differently based on the context provided by the concatenated text.
    
- ✅ double check chatgpt’s answer
    - look at  [Transformer from Scratch (solutions).ipynb](https://colab.research.google.com/drive/1qonznjskigQl59M78iIvj0hrKUb2CuUv#scrollTo=8XmSN1YFm4R5). It says to just concat the token. But verify this with more sources
    - Look at how huggingface’s `.generate` works.
        
        [https://github.com/huggingface/transformers/blob/v4.41.2/src/transformers/generation/utils.py#L1440](https://github.com/huggingface/transformers/blob/v4.41.2/src/transformers/generation/utils.py#L1440)
        
    - where in this code does it put the new token into the prompt again to refeed it?
        
        In the provided `generate` method for the transformer model, the newly generated tokens are re-fed into the model during the auto-regressive generation process. This occurs in the parts of the code where the method handles the sequence generation step by step, specifically in the `self._sample`, `self._beam_search`, or similar methods depending on the generation mode. Here’s a breakdown of how this happens:
        
        1. **Initial Setup:** The initial input tokens are prepared, and model configurations are set up.
        2. **Expansion and Preparation:** The `input_ids` tensor, which contains the initial tokens, is prepared and potentially expanded if using techniques like beam search.
        3. **Generation Loop:** The generation happens inside methods like `_sample` or `_beam_search`, where tokens are generated one by one and appended to the sequence. This loop is where the generated tokens are repeatedly added to the input and re-fed into the model to predict the next token. Here is where the key part of the auto-regressive process happens:
            
            ```python
            result = self._sample(
                input_ids,
                logits_processor=prepared_logits_processor,
                logits_warper=prepared_logits_warper,
                stopping_criteria=prepared_stopping_criteria,
                generation_config=generation_config,
                synced_gpus=synced_gpus,
                streamer=streamer,
                **model_kwargs,
            )
            
            ```
            
        4. **Auto-Regressive Generation:** Inside the `_sample` or `_beam_search` methods, the newly generated token is appended to `input_ids` and passed back into the model to generate the next token. This is a loop that continues until the stopping criteria are met (e.g., max length reached or end-of-sequence token generated).
        
        ### Explanation with Code References
        
        - **_prepare_model_inputs:** Sets up the initial model inputs.
        - **_sample / _beam_search:** Handles the iterative process of generating tokens and feeding them back into the model.
        - **input_ids Expansion:** The `input_ids` tensor is updated with new tokens during each step of generation.
        
        Here’s a simplified conceptual example of what happens in the loop (inside `_sample` method):
        
        ```python
        for step in range(max_length):
            model_outputs = self.model(input_ids, **model_kwargs)
            next_token_logits = model_outputs.logits[:, -1, :]
            next_token = torch.argmax(next_token_logits, dim=-1, keepdim=True)
            input_ids = torch.cat([input_ids, next_token], dim=-1)
        
        ```
        
        In this loop:
        
        - `input_ids` is initially the prompt provided.
        - For each step, the model generates logits for the next token.
        - The next token is selected and appended to `input_ids`.
        - This updated `input_ids` is used as input for the next iteration.
        
        This process ensures that the generated sequence is built token by token, with each new token being immediately re-fed into the model until the sequence is complete.
        
        ### References:
        
        - [GPT-2 Tokenization Process](https://lukesalamone.github.io/)【10†source】
        - [Byte Pair Encoding Walkthrough](https://hundredblocks.github.io/)【13†source】
        - [NLG with GPT-2](https://jaketae.github.io/)【12†source】
    - does that code concat the tokens?
        
        Yes, the code does concatenate the generated tokens to the existing sequence during the auto-regressive generation process. Specifically, this concatenation happens within the loop where new tokens are generated and appended to the `input_ids` tensor, which is then fed back into the model to predict the next token. This process is repeated until the stopping criteria are met.
        
        Here's a more detailed look at where this occurs in the code:
        
        ### Concatenation in the Generation Loop
        
        In the `_sample` method (and similar methods like `_beam_search`), the concatenation happens in each iteration of the loop:
        
        ```python
        for step in range(max_length):
            # Generate logits for the next token
            model_outputs = self.model(input_ids, **model_kwargs)
            next_token_logits = model_outputs.logits[:, -1, :]
        
            # Select the next token
            next_token = torch.argmax(next_token_logits, dim=-1, keepdim=True)
        
            # Concatenate the next token to the input_ids
            input_ids = torch.cat([input_ids, next_token], dim=-1)
        
        ```
        
        ### Explanation
        
        1. **Model Outputs:** The model generates logits for the next token based on the current `input_ids`.
        2. **Next Token Selection:** The next token is selected from the logits.
        3. **Concatenation:** The selected token (`next_token`) is concatenated to the existing sequence of `input_ids`.
        
        This concatenation (`torch.cat([input_ids, next_token], dim=-1)`) is critical as it extends the input sequence with each new token generated, allowing the model to use the entire sequence generated so far to inform the prediction of the next token.
        
        ### References
        
        - **GPT-2 Tokenization Process:** This source explains the process of tokenization and how sequences are handled during generation【10†source】【12†source】.
        - **Byte Pair Encoding Walkthrough:** Provides an overview of how tokens are generated and concatenated in practice【13†source】.
        
        By concatenating tokens in this way, the model ensures that it generates coherent sequences where each token is influenced by all previously generated tokens. This is a fundamental aspect of how transformer models perform auto-regressive generation.
        
    - Doesn't this go against what you said about how the model retokenizes?
        
        You're right; I need to clarify the process to ensure it aligns with what I previously explained about re-tokenization. The core idea remains that the model processes sequences token by token, and while the concatenation happens within the tensor operations during generation, the re-tokenization occurs when you re-feed text into the model as new input after decoding.
        
        ### Clarification of the Process
        
        1. **Auto-Regressive Token Generation:**
            - In the generation loop, tokens are concatenated to the existing `input_ids` tensor. This happens within a single call to the model's generation function.
            - The `input_ids` tensor grows with each new token, allowing the model to use the entire generated sequence so far to predict the next token.
        2. **Re-Feeding and Re-Tokenization:**
            - If you were to take the generated sequence, convert it back to text (decode it), and then feed this text as a new input to the model, the text would be re-tokenized from scratch.
            - This new tokenization can produce different token splits due to the context provided by the concatenated text.
        
        ### Detailed Example
        
        1. **Within One Generation Call (No Re-Tokenization):**
            - **Initial Input:** `"The dog"`
            - **Generation Step 1:** `" jumps"` (token for " jumps" added)
                - `input_ids` now contains tokens for `"The dog jumps"`
            - **Generation Step 2:** `" over"` (token for " over" added)
                - `input_ids` now contains tokens for `"The dog jumps over"`
            
            This concatenation continues within the tensor operations during the generation loop.
            
        2. **New Input with Re-Tokenization:**
            - **Generated Sequence:** `"The dog jumps over the lazy dog"`
            - **Decode to Text:** `decoded_text = tokenizer.decode(input_ids)`
            - **New Input:** `"The dog jumps over the lazy dog and runs away"`
            - **Re-Tokenization:** The new input text `"The dog jumps over the lazy dog and runs away"` is tokenized again, which can produce different tokens splits compared to the previous `input_ids` tensor.
        
        ### Code Example Clarification
        
        ```python
        for step in range(max_length):
            model_outputs = self.model(input_ids, **model_kwargs)
            next_token_logits = model_outputs.logits[:, -1, :]
            next_token = torch.argmax(next_token_logits, dim=-1, keepdim=True)
            input_ids = torch.cat([input_ids, next_token], dim=-1)  # Concatenation happens here
        
        ```
        
        ### Re-Tokenization Scenario
        
        When you take the final generated text and feed it back as a new input, you would do something like:
        
        ```python
        # Assuming final_text is the generated text sequence
        new_input_text = final_text + " and runs away"
        new_input_ids = tokenizer.encode(new_input_text)  # This is where re-tokenization happens
        
        ```
        
        ### References
        
        - **GPT-2 Tokenization Process:** Describes how tokenization and sequence generation are handled in GPT-2【10†source】【11†source】【12†source】.
        - **Byte Pair Encoding Walkthrough:** Explains the tokenization algorithm and its effects on sequences【13†source】【14†source】.
        
        In summary, the concatenation of tokens within the tensor operations during a single generation call does not involve re-tokenization. Re-tokenization occurs when the generated text is converted back to text and fed as a new input to the model.
        
    - so that .generate code doesn't use re-tokenization?
        
        The `.generate` method in transformer models like GPT-2 and LLaMA2 does not use re-tokenization within a single generation process. Instead, it continuously concatenates the newly generated tokens to the existing sequence of tokens (in the form of `input_ids`) without converting them back to text and re-tokenizing. This is a crucial distinction because re-tokenization refers to the process of taking text, splitting it into tokens, and then feeding those tokens into the model.
        
        ### Detailed Explanation:
        
        1. **Auto-Regressive Generation**: During the auto-regressive generation process, the model generates one token at a time. Each generated token is appended to the current sequence of tokens (`input_ids`), which is then used to generate the next token. This happens iteratively until the stopping criteria are met (e.g., reaching a maximum length or generating an end-of-sequence token).
        2. **Concatenation of Tokens**: The concatenation of tokens happens in the generation loop. Here, the newly generated token is added to the existing sequence of tokens (`input_ids`), effectively making the sequence longer with each iteration.
        3. **Re-Tokenization**: This occurs only if you take the entire generated sequence, convert it back to text, and then feed this text back into the model as a new input. This would involve decoding the sequence of tokens to text and then re-encoding (tokenizing) this text to a new sequence of tokens.
        
        ### Code Breakdown:
        
        - **During Generation**:
            
            ```python
            for step in range(max_length):
                model_outputs = self.model(input_ids, **model_kwargs)
                next_token_logits = model_outputs.logits[:, -1, :]
                next_token = torch.argmax(next_token_logits, dim=-1, keepdim=True)
                input_ids = torch.cat([input_ids, next_token], dim=-1)  # Concatenation happens here
            
            ```
            
        - **Re-Tokenization Scenario** (separate from the `generate` method):
            
            ```python
            # Assuming final_text is the generated text sequence
            new_input_text = final_text + " and runs away"
            new_input_ids = tokenizer.encode(new_input_text)  # This is where re-tokenization happens
            
            ```
            
        
        ### References:
        
        1. **GPT-2 Tokenization Process**: Discusses the continuous concatenation of tokens during generation and how re-tokenization only happens when converting text back to tokens【10†source】【11†source】.
        2. **Byte Pair Encoding (BPE) Walkthrough**: Provides insights into tokenization methods like BPE and how they handle sequences【13†source】【14†source】.
        3. **NLG with GPT-2**: Explains the generation process and how tokens are managed during this process【12†source】.
        
        In summary, the `.generate` method does not involve re-tokenization within the same generation call. Re-tokenization only occurs when the generated text is fed back into the model as a new input, involving a conversion from text to tokens.
        
- ✅ rename “llama2_ablate_prompts” to “llama2_ablate_prompts_explora_v1.ipynb”
- ✅ llama2_ablate_prompts_explora_v2.ipynb: uses zero ablation
    
    [llama2_ablate_prompts_explora_v2.ipynb](../Expm%20Results-%20EMNLP%20df8b503260cf48d987a58d25103b6638/llama2_ablate_prompts_explora_v2%20ipynb%20034368629f3e40869a4bc34a49180a49.md) : Concat tokens (not retokenize from adding to string) for seqcont word problems
    
    - ✅ Ablate top heads found by [Llama2_JanFebMar](https://colab.research.google.com/drive/1xRkMdXy32WeCyKWEkjd296kd8e5plk1J#scrollTo=susSZdqpqVzd).ipynb, then test:  [What are the months in a year?](../Expm%20Results-%20EMNLP%20df8b503260cf48d987a58d25103b6638/llama2_ablate_prompts_explora_v2%20ipynb%20034368629f3e40869a4bc34a49180a49.md)
- ✅ circuit for numerals 1 to 12
    
    start time: 943 ; end 1245
    
- ✅ v3: test numeral ‘more data’ impt heads on prompts
    
    [llama2_ablate_prompts_explora_v3.ipynb](../Expm%20Results-%20EMNLP%20df8b503260cf48d987a58d25103b6638/llama2_ablate_prompts_explora_v3%20ipynb%2085fbfa1671f644ae8dfcc4cd49b10d71.md) 
    
- ✅ ensure random doesn’t have 10 of the same heads from the circ (we allow other combos)