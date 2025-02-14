# API4S
Aqui está um guia para criar um projeto Spring Boot para o backend e Vue.js para o frontend, seguindo suas diretrizes.

---

## **1. Configuração do Backend (Spring Boot)**
### **1.1 Criar o Projeto Spring Boot**
Use o [Spring Initializr](https://start.spring.io/) ou crie manualmente com:
```bash
mvn archetype:generate -DgroupId=com.exemplo -DartifactId=backend -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```

**Dependências:**
- Spring Web
- Spring Data JPA
- Spring Security
- Spring Boot DevTools
- Lombok
- H2 Database (ou MySQL, se preferir)
- JWT (jjwt)

### **1.2 Estrutura de Pacotes**
```plaintext
backend/
 ├── src/main/java/com/exemplo
 │   ├── config/
 │   │   ├── CorsConfig.java
 │   │   ├── SecurityConfig.java
 │   │   ├── JwtUtil.java
 │   │   ├── JwtFilter.java
 │   ├── controller/
 │   │   ├── UsuarioController.java
 │   │   ├── ProdutoController.java
 │   ├── service/
 │   │   ├── UsuarioService.java
 │   │   ├── ProdutoService.java
 │   ├── repository/
 │   │   ├── UsuarioRepository.java
 │   │   ├── ProdutoRepository.java
 │   ├── model/
 │   │   ├── Usuario.java
 │   │   ├── Produto.java
 │   ├── dto/
 │   │   ├── UsuarioDTO.java
 │   │   ├── ProdutoDTO.java
 │   ├── BackendApplication.java
 ├── src/main/resources/
 │   ├── application.properties
 ├── pom.xml
```

### **1.3 Configuração do `application.properties`**
```properties
spring.datasource.url=jdbc:mysql://localhost:3306/meubanco
spring.datasource.username=root
spring.datasource.password=1234
spring.jpa.hibernate.ddl-auto=none  # O banco será criado manualmente
spring.jpa.show-sql=true
server.port=8080
```

### **1.4 Modelos**
```java
@Entity
public class Produto {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String nome;
    private Double preco;
}
```

```java
@Entity
public class Usuario {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String nome;
    private String email;
    private String senha;
}
```

### **1.5 Repositórios**
```java
@Repository
public interface ProdutoRepository extends JpaRepository<Produto, Long> {
    List<Produto> findByNomeContaining(String nome);
}
```

### **1.6 Serviços**
```java
@Service
public class ProdutoService {
    @Autowired
    private ProdutoRepository repository;

    public List<Produto> listar() {
        return repository.findAll();
    }

    public Produto salvar(Produto produto) {
        return repository.save(produto);
    }

    public void deletar(Long id) {
        repository.deleteById(id);
    }
}
```

### **1.7 Controladores**
```java
@RestController
@RequestMapping("/api/produtos")
@CrossOrigin(origins = "*")
public class ProdutoController {
    @Autowired
    private ProdutoService service;

    @GetMapping
    public List<Produto> listar() {
        return service.listar();
    }

    @PostMapping
    public Produto salvar(@RequestBody Produto produto) {
        return service.salvar(produto);
    }

    @DeleteMapping("/{id}")
    public void deletar(@PathVariable Long id) {
        service.deletar(id);
    }
}
```

---

## **2. Configuração do Frontend (Vue.js)**

### **2.1 Criar o Projeto Vue.js**
```bash
npm create vite@latest frontend --template vue
cd frontend
npm install
```

### **2.2 Instalar Dependências**
```bash
npm install axios vue-router vuex
```

### **2.3 Estrutura do Projeto**
```plaintext
frontend/
 ├── src/
 │   ├── assets/
 │   ├── components/
 │   │   ├── ProdutoLista.vue
 │   │   ├── ProdutoForm.vue
 │   ├── views/
 │   │   ├── Home.vue
 │   │   ├── Produtos.vue
 │   ├── router/
 │   │   ├── index.js
 │   ├── store/
 │   │   ├── index.js
 │   ├── App.vue
 │   ├── main.js
 ├── public/
 ├── package.json
```

### **2.4 Configuração do Vue Router (`router/index.js`)**
```javascript
import { createRouter, createWebHistory } from 'vue-router';
import Home from '../views/Home.vue';
import Produtos from '../views/Produtos.vue';

const routes = [
    { path: '/', component: Home },
    { path: '/produtos', component: Produtos },
];

const router = createRouter({
    history: createWebHistory(),
    routes,
});

export default router;
```

### **2.5 Configuração do Vuex (`store/index.js`)**
```javascript
import { createStore } from 'vuex';
import axios from 'axios';

export default createStore({
    state: {
        produtos: []
    },
    mutations: {
        SET_PRODUTOS(state, produtos) {
            state.produtos = produtos;
        }
    },
    actions: {
        async fetchProdutos({ commit }) {
            try {
                const response = await axios.get('http://localhost:8080/api/produtos');
                commit('SET_PRODUTOS', response.data);
            } catch (error) {
                console.error("Erro ao buscar produtos", error);
            }
        }
    }
});
```

### **2.6 Tela de Produtos (`views/Produtos.vue`)**
```vue
<template>
    <div>
        <h1>Lista de Produtos</h1>
        <button @click="carregarProdutos">Carregar Produtos</button>
        <ul>
            <li v-for="produto in produtos" :key="produto.id">
                {{ produto.nome }} - R$ {{ produto.preco }}
            </li>
        </ul>
    </div>
</template>

<script>
import { mapState, mapActions } from 'vuex';

export default {
    computed: {
        ...mapState(['produtos']),
    },
    methods: {
        ...mapActions(['fetchProdutos']),
        carregarProdutos() {
            this.fetchProdutos();
        }
    }
};
</script>
```

### **2.7 Configuração do Axios (`main.js`)**
```javascript
import { createApp } from 'vue';
import App from './App.vue';
import router from './router';
import store from './store';
import axios from 'axios';

axios.defaults.baseURL = 'http://localhost:8080/api';

createApp(App)
    .use(router)
    .use(store)
    .mount('#app');
```

### **2.8 Diretivas Vue.js**
- `v-bind`: Ligação dinâmica de atributos (`<img v-bind:src="imagem">`)
- `v-on`: Eventos (`<button v-on:click="salvar">Salvar</button>`)
- `v-model`: Bind bidirecional (`<input v-model="nome">`)
- `v-if`: Condições (`<p v-if="mostrar">Olá</p>`)
- `v-for`: Loops (`<li v-for="item in lista" :key="item.id">{{ item.nome }}</li>`)

---

## **3. Testando o Projeto**
1. **Suba o backend:**
   ```bash
   mvn spring-boot:run
   ```
2. **Suba o frontend:**
   ```bash
   npm run dev
   ```

Agora você tem um CRUD básico com Spring Boot e Vue.js, seguindo todas as diretrizes que você solicitou. 🚀
