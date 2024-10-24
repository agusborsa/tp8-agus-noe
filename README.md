# Auditoría del código de Ro

## Cross-Site Scripting (XSS) en generateProfileHtml
### Tipo: OWASP Top 10 2021: A03 - Cross-Site Scripting (XSS)

### Prueba de Concepto
- A continuación, el fragmento de código inseguro:
```javascript
const obscureHtml = (str) => {
    return str.split('').map(char => char + '').join('');
};

const reassembleHtml = (part1, part2, part3) => {
    return obscureHtml(part1) + obscureHtml(part2) + obscureHtml(part3);
};

const generateProfileHtml = (user) => {
    const { username, role } = user;
    let part1 = `<h1>Pro` + `file of `;
    let part2 = `${username}</h1>`;             //<---- aqui ocurriría el XXS
    let part3 = `<p>Role: ${role}</p>`;
    return reassembleHtml(part1, part2, part3);
};

const sendProfile = (res, user) => {
    const html = generateProfileHtml(user);
    res.send(html);
};
```
### Se podría inyectar HTML desde el input de user.
```javascript
const user = {
  username: '<script>alert("XSS")</script>',
  role: 'admin'
};

const html = generateProfileHtml(user);
console.log(html); 
```

### Salida esperada:
```html
<h1>Profile of <script>alert("XSS")</script></h1><p>Role: admin</p>
```

### Riesgo: Medio
Actualmente no se está utilizando la estructura, por lo que el riesgo en sí es "medio" pero si se escribió pensando en su futura implementación en producción, un atacante podría inyectar código malicioso, como scripts, en los campos username o role, los cuales se reflejan directamente en la página HTML generada. Esto puede permitir la ejecución de scripts maliciosos en el navegador de los usuarios, comprometiendo su información o redirigiéndolos a sitios maliciosos.

### Recomendaciones
- Es necesario sanitizar cualquier entrada del usuario antes de incrustarla en el HTML. Se recomienda usar una función de sanitización para escapar caracteres especiales:

```javascript
const sanitizeHtml = (str) => {
    return str.replace(/</g, "&lt;").replace(/>/g, "&gt;");
};

let part2 = `${sanitizeHtml(username)}</h1>`;
let part3 = `<p>Role: ${sanitizeHtml(role)}</p>`;
```

- Esto evitará que los atacantes inyecten scripts maliciosos en los valores de los campos.
- También se puede remover la estructura directamente si no está pensada para ser utilizada en un futuro!