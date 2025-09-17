# Relatório: Mini-Projeto 1 - Quebra-Senhas Paralelo

**Aluno(s):** Nome (Matrícula), Pedro Henrique Carvalho Pereira (10418861),,,  
---

## 1. Estratégia de Paralelização


**Como você dividiu o espaço de busca entre os workers?**

[Explique seu algoritmo de divisão]

**Código relevante:** Cole aqui a parte do coordinator.c onde você calcula a divisão:
```c
// Cole seu código de divisão aqui
```

---

## 2. Implementação das System Calls

**Descreva como você usou fork(), execl() e wait() no coordinator:**

[Explique em um parágrafo como você criou os processos, passou argumentos e esperou pela conclusão]

**Código do fork/exec:**
```c
// Cole aqui seu loop de criação de workers
```

---

## 3. Comunicação Entre Processos

**Como você garantiu que apenas um worker escrevesse o resultado?**

A escrita atômica foi feita a partir dos parametros O_CREAT | O_EXCL, passados para a função open(). A junção desses parametros protege a função contra a tentativa de dois ou mais workers criarem o arquivo ao mesmo tempo garantimos que o arquivo será criado uma única vez. 
O primeiro worker cria o arquivo e os seguintes, se tentarem criar, recebem erro ('fd < 0' e 'errno == EEXIST'), garantindo a não sobrescrita. 
Considerando uma condição de corrida toda vez que o resultado depende da ordem de execução das threads e observando que quando o primeiro worker chega, cria e escreve no arquivo e os outros falham ao tentar criar o mesmo arquivo, isso garante a exclusividade de apenas um worker escrever o resultado (sem necessidade de mutex, variáveis de condição ou semáforos). 
Leia sobre condições de corrida (aqui)[https://pt.stackoverflow.com/questions/159342/o-que-%C3%A9-uma-condi%C3%A7%C3%A3o-de-corrida]

**Como o coordinator consegue ler o resultado?**

O coordinator abre o arquivo de resultado (caso ele exista). 
Lê o conteúdo em um buffer. Nesse caso lê até 255 bytes e adciona \0 no final pra converter em string válida (buffer[n] = '\0';). Assim, o buffer vai conter uma string com o formato "worker_id:passowrd".
Depois é feito o parse do formato (worker_id:passowrd) usando o sscanf (sscanf(buffer, "%d:%32s", &worker_id, found_password) == 2) -> sscanf retorna dois se o parse funcionar. 
Depois verifica-se o hash da senha encontrada usando 'md5_string', garantindo corretude do resultado. 
Por fim, é exibida a senha e o ID do worker que encontrou o resultado. E fecha o arquivo.

---

## 4. Análise de Performance
Complete a tabela com tempos reais de execução:
O speedup é o tempo do teste com 1 worker dividido pelo tempo com 4 workers.

| Teste | 1 Worker | 2 Workers | 4 Workers | Speedup (4w) |
|-------|----------|-----------|-----------|--------------|
| Hash: 202cb962ac59075b964b07152d234b70<br>Charset: "0123456789"<br>Tamanho: 3<br>Senha: "123" | 0.027s | 0.024s | 0.021s | 1.29x |
| Hash: 5d41402abc4b2a76b9719d911017c592<br>Charset: "abcdefghijklmnopqrstuvwxyz"<br>Tamanho: 5<br>Senha: "hello" | 2.179s | 2.297s | 0.292s | 7.46x |

**O speedup foi linear? Por quê?**
Para o teste usando a senha '123', quando dobramos o número de workers (1 -> 2 -> 4), nota-se uma redução no tempo, mas uma redução pequena, com speedup calculado em 1.29x. Isso acontece porque a senha é pequena, ou seja, o espaço de busca é pequeno e o overhead da criação dos processos com fork/exec consomem tempo significativo, por isso não houveram grandes ganhos.
Já no caso da senha 'hello', o espaço de busca é muito maior, então o aumento dos workers impactou positivamente na busca pela senha correta. De 1 worker para 2 até houve um aumento, provavelmente devido à sincronização e criações de processos, mas quando testamos para 4 workers, o tempo caiu em aproximadamento 2s, resultando em um speedup de 7.46x, mas ainda sim não foi exatamente linear.
Em suma, podemos concluir que o speedup tende a ser mais próximo do linear quando a senha é grande e bem dividida entre os workers, já em problemas pequenos ou com overhead significativo, o speedup não é linear.
---

## 5. Desafios e Aprendizados
**Qual foi o maior desafio técnico que você enfrentou?**
[Descreva um problema e como resolveu. Ex: "Tive dificuldade com o incremento de senha, mas resolvi tratando-o como um contador em base variável"]
Pedro (10418861): Minha maior dificuldade foi chegar na implementação do TODO 9. Para solucionar e avançar no projeto, pesquisei sobre métodos de leitura atômica de arquivos (para evitar conflito entre threads), leitura de arquivo em buffer e sobre o método de parsing usando o sscanf limitando o tamanho da senha para evitar overflow.
---

## Comandos de Teste Utilizados

```bash
# Teste básico
./coordinator "900150983cd24fb0d6963f7d28e17f72" 3 "abc" 2

# Teste de performance
time ./coordinator "202cb962ac59075b964b07152d234b70" 3 "0123456789" 1
time ./coordinator "202cb962ac59075b964b07152d234b70" 3 "0123456789" 4

# Teste com senha maior
time ./coordinator "5d41402abc4b2a76b9719d911017c592" 5 "abcdefghijklmnopqrstuvwxyz" 4
```
---

**Checklist de Entrega:**
- [X] Código compila sem erros
- [X] Todos os TODOs foram implementados
- [X] Testes passam no `./tests/simple_test.sh`
- [ ] Relatório preenchido
