# dio-app-com-fastapi-colab
## Criação de uma aplicação FastAPI em Python a nível de demonstração no Colab

Esse projeto demonstra de modo simples o uso do FastAPI no modo assíncrono no Google Colab. Inclusive, o FastAPI é construído com base em asyncio ( a biblioteca de programação assíncrona nativa do Python ) cuja documentação está disponível em https://docs.python.org/3/library/asyncio.html, o que o torna ideal para lidar com requisições concorrentes e operações de I/O de forma eficiente. No entanto, existem algumas considerações e passos adicionais para fazer isso funcionar corretamente no Colab, que aprendi após algumas pesquisas e erros tomados no Colab:
1. **Colab e Servidores Web**: O Google Colab é um ambiente de notebook e não foi projetado para rodar servidores web diretamente na porta padrão. Por isso, você precisará de uma ferramenta como o **ngrok**, a qual faço já uso há 4 anos e recomendo quem não usa procurar saber mais em https://ngrok.com/docs, para expor sua aplicação FastAPI na internet, gerando uma **URL** pública.
2. `nest_asyncio`: Jupyter notebooks (incluindo o Colab) já executam um loop de eventos `asyncio` em segundo plano. Tentar iniciar um novo loop com `uvicorn.run()` diretamente pode gerar um erro _RuntimeError: asyncio.run() cannot be called from a running event loop_. Para contornar isso, você pode usar a biblioteca `nest_asyncio`, que permite o uso aninhado de `asyncio.run()`.

### Passos para rodar FastAPI assíncrono no Colab:
- Instale as seguintes bibliotecas: `!pip install fastapi uvicorn nest-asyncio pyngrok -q`
- Configure o ngrok:
  * Em https://ngrok.com/ crie uma conta, inclusive usando suas credenciais do google.
  * Obtenha seu `authtoken` no dashboard do ngrok.
  * No seu notebook Colab, defina o `authtoken`:
  ```
  from pyngrok import ngrok
  authtoken = "SEU_AUTHTOKEN_AQUI"
  ngrok.set_auth_token(authtoken)
  ```
- Crie um App em FastAPI:
  ```
  from fastapi import FastAPI
  import asyncio
  import nest_asyncio
  import uvicorn
  
  app = FastAPI()
  
  @app.get("/async_data")
  async def get_async_data():
    await asyncio.sleep(5)
    return {"message": "Retorno: Dados assíncronos!"}
  
  @app.get("/hello")
  async def hello_world():
    return {"message": "Olá, esse é meu app em FastAPI assíncrono no Colab!"}
  ```
- Rode o servidor FastAPI com nest_asyncio e ngrok:
  ```
  nest_asyncio.apply()

  # Expor o servidor local para a internet usando ngrok
  public_url = ngrok.connect(8000)
  print("URL Pública do FastAPI:", public_url)
  
  # Rodar o Uvicorn. O loop de eventos já está em execução graças ao nest_asyncio.
  # Você pode rodar isso em uma thread separada ou usar a função Server para mais controle.
  # Uma forma comum é rodar em uma thread para não bloquear a execução do notebook:
  import threading
  def run_uvicorn():
    uvicorn.run(app, host="0.0.0.0", port=8000)
  
  thread = threading.Thread(target=run_uvicorn)
  thread.start()
  
  # O servidor estará rodando em segundo plano.
  # Você pode agora acessar a URL pública fornecida pelo ngrok.
  ```
- O ngrok fornecerá uma URL pública, que você poderá usar para acessar seus endpoints do FastAPI.
- Ao usar async def em suas funções de operação de rota no FastAPI, você está indicando que essas funções são "corrotinas" e podem ser "pausadas" (await) enquanto esperam por uma operação demorada ser concluída, permitindo que o servidor processe outras requisições nesse meio tempo e resumidamente obtém: 
  * Concorrência: O FastAPI pode lidar com milhares de requisições simultaneamente sem bloquear.
  * I/O Eficiente: Ideal para aplicações que fazem muitas chamadas a APIs externas, interagem com bancos de dados ou realizam operações de leitura/escrita de arquivos, pois o await permite que outras operações sejam processadas enquanto a tarefa de I/O está esperando.
  * Escalabilidade: Reduz a necessidade de threads adicionais, levando a um menor uso de memória e maior capacidade de resposta.
