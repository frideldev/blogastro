## **Informe: Manipulación de Archivos Markdown para Publicaciones en un Blog**

### **1. Introducción**

Este informe detalla cómo utilizar archivos Markdown para almacenar y gestionar publicaciones en un blog desarrollado con Astro, con la infraestructura backend desplegada en Vercel. Se cubren las operaciones básicas para la manipulación de estos archivos, incluyendo cómo se almacenan las publicaciones, cómo se muestran en el sitio web, y cómo se agregan nuevas publicaciones.

### **2. Estructura y Manipulación de Archivos Markdown**

#### **2.1. Estructura de Archivos Markdown**

Cada publicación de blog se almacena en un archivo Markdown (`.md`). La estructura básica de un archivo Markdown para una publicación es la siguiente:

```markdown
---
title: "Título de la Publicación"
date: "2024-08-29"
description: "Descripción breve de la publicación"
author: "Nombre del Autor"
---

Contenido de la publicación aquí...
```

- **Metadatos:** Encerrados entre triple guión (`---`), contienen información estructural de la publicación, como el título, la fecha, la descripción, y el autor.
- **Contenido:** El texto que sigue a los metadatos es el contenido principal de la publicación.

#### **2.2. Cómo Astro Maneja los Archivos Markdown**

Astro proporciona una forma eficiente de gestionar archivos de contenido mediante el módulo `astro:content`. Con este módulo, puedes cargar y mostrar publicaciones almacenadas en archivos Markdown.

1. **Listar Publicaciones:**

   En `src/pages/blog/index.astro`, puedes recuperar todas las publicaciones y mostrarlas en una lista:

   ```astro
   ---
   import { getCollection } from 'astro:content';

   const posts = await getCollection('blog');
   ---

   <Layout>
     <h1>Blog de Eco Turismo en Tarija</h1>
     <ul>
       {posts.map(post => (
         <li>
           <a href={`/blog/${post.slug}`}>{post.data.title}</a>
           <p>{post.data.description}</p>
         </li>
       ))}
     </ul>
   </Layout>
   ```

2. **Mostrar una Publicación Individual:**

   En `src/pages/blog/[slug].astro`, puedes cargar y mostrar una publicación específica usando su `slug`:

   ```astro
   ---
   import { getEntryBySlug } from 'astro:content';
   import Layout from '../../components/Layout.astro';

   const { slug } = Astro.params;
   const post = await getEntryBySlug('blog', slug);
   ---

   <Layout>
     <h1>{post.data.title}</h1>
     <p>Publicado el {post.data.date}</p>
     <p>Autor: {post.data.author}</p>
     <article>
       {post.content}
     </article>
   </Layout>
   ```

### **3. Agregar Nuevas Publicaciones**

#### **3.1. Crear un Formulario para Nuevas Publicaciones**

En `src/pages/admin/index.astro`, se puede crear un formulario para permitir a los usuarios agregar nuevas publicaciones:

```astro
---
import Layout from '../../components/Layout.astro';

const handleSubmit = async (event) => {
  event.preventDefault();

  const formData = new FormData(event.target);
  const newPost = {
    title: formData.get('title'),
    date: new Date().toISOString().split('T')[0],
    description: formData.get('description'),
    author: formData.get('author'),
    content: formData.get('content'),
    slug: formData.get('title').toLowerCase().replace(/\s+/g, '-'),
  };

  const response = await fetch('/api/savePost', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(newPost),
  });

  if (response.ok) {
    alert('Publicación guardada con éxito');
  } else {
    alert('Error al guardar la publicación');
  }
};
---

<Layout>
  <h1>Crear Nueva Publicación</h1>
  <form onSubmit={handleSubmit}>
    <div>
      <label for="title">Título:</label>
      <input type="text" name="title" required />
    </div>
    <div>
      <label for="description">Descripción:</label>
      <input type="text" name="description" required />
    </div>
    <div>
      <label for="author">Autor:</label>
      <input type="text" name="author" required />
    </div>
    <div>
      <label for="content">Contenido:</label>
      <textarea name="content" required></textarea>
    </div>
    <button type="submit">Guardar</button>
  </form>
</Layout>
```

#### **3.2. Crear una Función API para Guardar Publicaciones**

Para manejar el almacenamiento de las publicaciones en archivos Markdown, se debe implementar una función API en Vercel. Aquí un ejemplo en `api/savePost.js`:

```javascript
import { promises as fs } from 'fs';
import path from 'path';

export default async function handler(req, res) {
  if (req.method === 'POST') {
    const post = req.body;
    const filePath = path.join(process.cwd(), 'src/content/blog', `${post.slug}.md`);

    const markdownContent = `---
title: "${post.title}"
date: "${post.date}"
description: "${post.description}"
author: "${post.author}"
---

${post.content}`;

    try {
      await fs.writeFile(filePath, markdownContent);
      res.status(200).json({ message: 'Publicación guardada con éxito' });
    } catch (error) {
      console.error(error);
      res.status(500).json({ error: 'Error al guardar la publicación' });
    }
  } else {
    res.setHeader('Allow', ['POST']);
    res.status(405).end(`Method ${req.method} Not Allowed`);
  }
}
```

- **`fs.writeFile(filePath, markdownContent)`** se utiliza para escribir el archivo Markdown con el contenido proporcionado.

### **4. Consideraciones Adicionales**

1. **Despliegue en Vercel:**
   - Asegúrate de desplegar tanto el proyecto de Astro como la función API en Vercel para que estén disponibles para su uso.

2. **Automatización de Construcción:**
   - Considera configurar un webhook en Vercel para que se realice una reconstrucción automática del sitio cuando se realicen cambios en las publicaciones.

3. **Pruebas y Verificación:**
   - Realiza pruebas exhaustivas para asegurar que el formulario de administración y la función API funcionen correctamente, y que las publicaciones se reflejen en el sitio web de manera adecuada.

---

Este informe proporciona una visión completa de cómo manejar archivos Markdown en un blog con Astro y Vercel, desde la estructura de los archivos hasta la adición y almacenamiento de nuevas publicaciones. ¿Hay algún detalle adicional que te gustaría agregar o algún aspecto específico que necesites profundizar?
