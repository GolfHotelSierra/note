[toc]

> [official code](https://github.com/rinongal/textual_inversion)

# Preliminary

## CLIPTextModel 的结构

- CLIPTextModel 是 sd 中默认使用的 text encoder，textual inversion 的实现基于对 CLIPTextModel 的<u>*源码的修改*</u>
- FrozenCLIPEmbedder - (1) CLIPTokenizer (2) CLIPTextModel - CLIPTextTransformer - (1) CLIPTextEmbeddings (2) CLIPEncoder
  - FrozenCLIPEmbedder - (2) CLIPTextModel - CLIPTextTransformer 可以简单视作外层的封装
  - CLIPTokenizer 用于分词，对于 1 个 token 一般输出 1 个 <u>*1 维的 id*</u>
  - CLIPTextEmbeddings 将 1 维的 id <u>*提升为更高维度*</u> (512, 768 e.t.c.)；CLIPEncoder 中实例化了一个 <u>*transformer*</u> 结构

> 注意，FrozenCLIPEmbedder 里的 CLIPTextModel 实例叫做 self.transformer；CLIPTextModel 里的 CLIPTextTransformer 实例叫做 self.text_model，名字并不是一一对应的





# Textual Inversion

## 项目结构

- 默认的网络结构配置文件为 `textual_inversion/configs/stable-diffusion/v1-inference.yaml`
- 整体结构<u>*与 sd 1.4 基本一致*</u>，直接在源码基础上做修改



## `EmbeddingManager` - 管理 placeholder 的类

> textual_inversion-main\ldm\modules\embedding_manager.py

**`__init__()`**

```python
class EmbeddingManager(nn.Module):

    def __init__(
            self,
            embedder,
            placeholder_strings=None,
            initializer_words=None,
            ...
    ):
        # :param embedder: 使用的文本嵌入模型(CLIP e.t.c.)
		# :param placeholder_strings: 占位符字符串列表，用于表示需要被替换的特定标记
		# :param initializer_words: 初始化单词列表，用于初始化占位符的嵌入; 不提供则随机初始化

        super().__init__()
		# placeholder_string -> 分词结果
        self.string_to_token_dict = {}
        # placeholder_string -> embedding
        self.string_to_param_dict = nn.ParameterDict()
        self.initial_embeddings = nn.ParameterDict()

        ...
        
        # 将CLIPTokenizer的分词方法绑定到get_token_for_string()上, 将CLIPEmbedding的嵌入方法绑定到get_embedding_for_tkn()上
        if hasattr(embedder, 'tokenizer'):
            self.is_clip = True
            get_token_for_string = partial(get_clip_token_for_string, embedder.tokenizer)
            get_embedding_for_tkn = partial(get_embedding_for_clip_token, embedder.transformer.text_model.embeddings)
            token_dim = 768
        else: # using LDM's BERT encoder
            # ...

        # ...

        for idx, placeholder_string in enumerate(placeholder_strings):          
            
            token = get_token_for_string(placeholder_string)
            
            if initializer_words and idx < len(initializer_words):
                init_word_token = get_token_for_string(initializer_words[idx])
                
                with torch.no_grad():
                    # 根据initializer_words获取初始化的embedding
                    init_word_embedding = get_embedding_for_tkn(init_word_token.cpu())

                # 这里还要过一遍nn.Parameter()猜测是原本的类型不能梯度回传?
                token_params = torch.nn.Parameter(init_word_embedding.unsqueeze(0).repeat(num_vectors_per_token, 1), requires_grad=True)
                # 用于计算损失函数, 所以梯度不回传
                self.initial_embeddings[placeholder_string] = torch.nn.Parameter(init_word_embedding.unsqueeze(0).repeat(num_vectors_per_token, 1), requires_grad=False)
            else:
                ...
            
            self.string_to_token_dict[placeholder_string] = token
            self.string_to_param_dict[placeholder_string] = token_params
```

**`__forward()__`**

```python
class EmbeddingManager(nn.Module):
    
    def forward(
            self,
            tokenized_text,
            embedded_text,
    ):
        # :param embedded_text: 包括placeholder的, CLIP计算的embedding结果
        # :return: 将替换完成的embedding返回

        # 这个for循环应该是表示支持一个prompt中出现多个placeholder
        for placeholder_string, placeholder_token in self.string_to_token_dict.items():

            placeholder_embedding = self.string_to_param_dict[placeholder_string].to(device)

            # 如果每个placeholder_token只对应一个vector，那么直接进行替换
            if self.max_vectors_per_token == 1:
                # 根据token id获取对应的索引
                placeholder_idx = torch.where(tokenized_text == placeholder_token.to(device))
                # 进行替换
                embedded_text[placeholder_idx] = placeholder_embedding
            # 如果对应多个vector, 比较复杂但逻辑一致
            else:
                # ...

        return embedded_text
```



## `EmbeddingManager` 实例化与调用

> textual_inversion/ldm/modules/encoders/modules.py

**`__init__()`**

- 因为多引入了 EmbeddingMananger 来对 placeholder 进行替换，所以按照之前提到的 FrozenCLIPEmbedder 中的各个结构，都需要被**修改 forward() 函数**
  - 如果是有实际计算逻辑的，则**增加调用 embedding manager 的逻辑**
  - 如果是封装类，则**在参数中额外传入 embedding manager**
- 具体代码使用 `__get__()` 方法**重新绑定了 forward() 函数**，简单说可以看做就是 `=` 赋值

> 具体代码的逻辑是每次经过 cond_stage_encoder 都会触发 embedding_manager，猜测是指向的都是同一个引用，所以梯度回传都是可以延续下去的

```python
class FrozenCLIPEmbedder(AbstractEncoder):

    def __init__(self, version="openai/clip-vit-large-patch14", device="cuda", max_length=77):
        super().__init__()
        self.tokenizer = CLIPTokenizer.from_pretrained(version)
        self.transformer = CLIPTextModel.from_pretrained(version)
        self.device = device
        self.max_length = max_length

        def embedding_forward(
                self,
                input_ids = None,
                position_ids = None,
                inputs_embeds = None,
            	# 增加embedding_manager参数
                embedding_manager = None,
            ) -> torch.Tensor:
				...
                
                # CLIPTextEmbeddings需要增加新的逻辑, 将placeholder换为经过训练的embedding
                if embedding_manager is not None:
                    inputs_embeds = embedding_manager(input_ids, inputs_embeds)

                position_embeddings = self.position_embedding(position_ids)
                embeddings = inputs_embeds + position_embeddings
                
                return embeddings      

        # 替换原来的forward()
        self.transformer.text_model.embeddings.forward = embedding_forward.__get__(self.transformer.text_model.embeddings)

        
        # CLIPEncoder的forward()也被替换了, 但好像没有做什么改动
        ...


        def text_encoder_forward(
            self,
            input_ids = None,
            attention_mask = None,
            position_ids = None,
            output_attentions = None,
            output_hidden_states = None,
            return_dict = None,
            # 增加embedding_manager参数
            embedding_manager = None,
        ):
            ...
            
            # 作为外层的封装类, 继续向内传递参数
            hidden_states = self.embeddings(input_ids=input_ids, position_ids=position_ids, embedding_manager=embedding_manager)

            ...

            last_hidden_state = self.encoder(
                inputs_embeds=hidden_states,
                attention_mask=attention_mask,
                causal_attention_mask=causal_attention_mask,
                output_attentions=output_attentions,
                output_hidden_states=output_hidden_states,
                return_dict=return_dict,
            )

            last_hidden_state = self.final_layer_norm(last_hidden_state)

            return last_hidden_state

        # 替换原来的forward()
        self.transformer.text_model.forward = text_encoder_forward.__get__(self.transformer.text_model)

        def transformer_forward(
            self,
            input_ids = None,
            attention_mask = None,
            position_ids = None,
            output_attentions = None,
            output_hidden_states = None,
            return_dict = None,
            # 增加embedding_manager参数
            embedding_manager = None,
        ):
            return self.text_model(
                input_ids=input_ids,
                attention_mask=attention_mask,
                position_ids=position_ids,
                output_attentions=output_attentions,
                output_hidden_states=output_hidden_states,
                return_dict=return_dict,
                # 继续向内传递
                embedding_manager = embedding_manager
            )
            
		# 替换原来的forward()
        self.transformer.forward = transformer_forward.__get__(self.transformer)
```



## 参数冻结

> textual_inversion-main\ldm\models\diffusion\ddpm.py

> 在 DDPM 的 `__init__()` 方法中有一个 `unfreeze_model=False` 参数，但是使用这个参数关闭梯度的逻辑却是在 LatentDiffusion 中，不知道为什么这么设计？

```python
class LatentDiffusion(DDPM):

    def __init__(self,
                 ...):

        ...

        # 关闭embedding_manager以外的梯度
        if not self.unfreeze_model:
            self.cond_stage_model.eval()
            self.cond_stage_model.train = disabled_train
            for param in self.cond_stage_model.parameters():
                param.requires_grad = False

            self.model.eval()
            self.model.train = disabled_train
            for param in self.model.parameters():
                param.requires_grad = False
        
        self.embedding_manager = self.instantiate_embedding_manager(personalization_config, self.cond_stage_model)

    	# 开启embedding_manager的梯度回传
        for param in self.embedding_manager.embedding_parameters():
            param.requires_grad = True
```

