# Redis

### É um banco de dados sem um schema, que salva apenas chave e valor é utilizado por exemplo para armazenar dados que será usados em um gerenciador de filas, sendo assim um banco extremamente rapido.

<bold> Instale por exemplo dentro do docker:</bold>
<pre>
docker run --name redisbarber -p 6379:6379 -d -t redis:alpine 
</pre>


#### Crie um arquivo redis.js no seu projeto na pasta config com a segunte configuração

```javascript
export default {
  host: '127.0.0.1',
  port: 6379,
};

```

### Na classe que contém a funcionalidade da fila importe o mesmmo e faça com a seguir
```javascript
import Bee from 'bee-queue';
import CancellationMail from '../app/jobs/CancellationMail';
import redisConfig from '../config/redis';

// new jobs will be pushed to this array;
const jobs = [CancellationMail];

class Queue {
  constructor() {
    this.queues = {};

    this.init();
  }

  // iniciamente para cada um do jobs cria uma filha e armazena o bee que contém a instancia com dados de conexão com o redis
  // e o meodo que handle que é o cara que processa a fila, seta as variáveis
  init() {
    // desestruturando para ter acesso direto a chave e handle metodo da classe de job CancellationMail
    jobs.forEach(({ key, handle }) => {
      this.queues[key] = {
        bee: new Bee(key, {
          redis: redisConfig,
        }),
        handle,
      };
    });
  }

  // armazena o job na fila
  add(queue, job) {
    return this.queues[queue].bee.createJob(job).save();
  }

  // processa o job em background
  processQueue() {
    jobs.forEach(job => {
      const { bee, handle } = this.queues[job.key];
      bee.on('failed',this.handleFailure).process(handle);
    });
  }
  
  handleFailure(job,err){
   console.log(`Queue ${job.queue.name} failed:  ${err}`);
  }
}

export default new Queue();

```
