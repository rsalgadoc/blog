---
layout: home
title: "Desafiando la Nube: 365 Días de AWS"
---

# ☁️ AWS Daily Insights
**Arquitectura, Automatización y Mejores Prácticas en la Nube.**

¡Hola! Soy Rodrigo Salgado, y he decidido documentar mi camino hacia la maestría en **Amazon Web Services**. Mi objetivo es claro: **un post nuevo cada día** compartiendo soluciones reales, scripts listos para producción y arquitecturas escalables.

---

### 🚀 El Reto: 1 Post al Día
Cada entrada incluye el **código fuente** (Terraform, Python/Boto3, CloudFormation) para que puedas replicar los laboratorios en tu propia cuenta.

#### 🛠️ Lo que encontrarás aquí:
*   **Serverless:** Profundizando en Lambda, API Gateway y DynamoDB.
*   **Infraestructura como Código (IaC):** Automatización total con Terraform y CDK.
*   **Seguridad:** Implementación de IAM, KMS y Shield siguiendo el *Well-Architected Framework*.
*   **Cost Optimization:** Estrategias para no llevarse sorpresas en la factura de AWS.

---

### 📚 Últimas Publicaciones
Aquí aparecerán mis posts más recientes. ¡Vuelve mañana para el siguiente!

<ul>
  {% for post in site.posts limit:5 %}
    <li>
      <a href="{{ post.url }}">{{ post.date | date: "%Y-%m-%d" }} - {{ post.title }}</a>
      <p>{{ post.excerpt | strip_html | truncatewords: 20 }}</p>
    </li>
  {% endfor %}
</ul>

---

### 🔗 Conecta conmigo
¿Tienes alguna duda sobre un servicio específico de AWS? 
[LinkedIn](www.linkedin.com/in/rodrigo-salgado-cordova) | [GitHub](https://github.com/rsalgadoc) | [Certificaciones](https://www.credly.com/users/rodrigo-salgado-cordova)
