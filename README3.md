   ### Пример безопасного RAG-паттерна
   def generate_response(query):
       relevant_data = vector_db.search(query)  # Поиск в защищенной БД
       return llm.generate(context=relevant_data, question=query)



   #### Пример санитизации ввода
   def sanitize_input(text: str) -> str:
       # Удаление специальных команд
       text = re.sub(r'(игнорируй|забудь|выполни).*инструкци', '[REDACTED]', text, flags=re.IGNORECASE)
       # Удаление URL
       text = re.sub(r'https?://\S+', '[URL REDACTED]', text)
       return text


  ### RAG с верификацией источников
     def generate_response(query):
         docs = vector_db.search(query, filter_by_trusted_sources=True)  # Только проверенные данные
         return llm.generate(context=docs)
