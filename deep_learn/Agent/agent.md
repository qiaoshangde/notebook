```
def create_llm(provider: Literal["openai", "anthropic", "gemini"]):


def connect(host: Optional[str] = None):


from abc import ABC, abstractmethod
抽象基类，不能实例化，定义子类必须实现的抽象方法。如果不是实现就会报错
用途如下：
class BaseLLMAdapter(ABC):  
  
    @abstractmethod  
    def invoke(self, messages):  
        pass  
  
    @abstractmethod  
    def stream_invoke(self, messages):  
        pass

子类：

class OpenAIAdapter(BaseLLMAdapter):  
  
    def invoke(self, messages):  
        ...  
  
    def stream_invoke(self, messages):  
        ...
        
```