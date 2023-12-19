# Izvješće 2 - Apstrakcija OpenGL-a

OpenGL je tehnička specifikacija, dok je OpenGL API njena implementacija. OpenGL API je napisan u C-u. OpenGL API je definiran kao stanje mašine. Skoro sve OpenGL funkcije postavljaju ili dohvaćaju neko stanje u OpenGL-u. Jedine funkcije koje ne mijenjaju stanje su funkcije koje koriste trenutno postavljeno stanje kako bi izazvale iscrtavanje. Možete zamisliti stanje mašine kao jako veliku strukturu s velikim brojem različitih polja. Ta struktura se zove OpenGL kontekst, a svako polje u kontekstu predstavlja neke informacije potrebne za iscrtavanje.

Na primjer ako bi za neki objekt koji sadrži integer, float i string htjeli pristupiti, u C++ bi izgledalo ovako:

```cpp
struct Object
{
    int count;
    float opacity;
    char *name;
};

//Create the storage for the object.
Object newObject;

//Put data into the object.
newObject.count = 5;
newObject.opacity = 0.4f;
newObject.name = "Object";
```

Dok bi u OpenGL izgledalo ovako:

```cpp
//Create the storage for the object
GLuint objectName;
glGenObject(1, &objectName);

//Put data into the object.
glBindObject(GL_MODIFY, objectName);
glObjectParameteri(GL_MODIFY, GL_OBJECT_COUNT, 5);
glObjectParameterf(GL_MODIFY, GL_OBJECT_OPACITY, 0.4f);
glObjectParameters(GL_MODIFY, GL_OBJECT_NAME, "Object");
```

Tako kod pisanja OpenGL koda, imamo dosta atributa koje moramo podesiti, ali isto tako treba voditi računa koji kontekst je vezan. Čak i kod iscrtavanja najosnovnijeg oblika poput trokuta, *Hello Triangle* aplikacije, potrebno je jako puno linija koda. Zbog velikog broja linija koda, dolazi do ružnog koda za čitanje, teškog snalaženja i održavanja. Apstrakcijom OpenGL funkcija dolazi do znatnog manjeg broja linija koda, te lakše nadogradnje aplikacije novim funkcionalnostima. 

OpenGL je apstraktiran po klasama:

1. **Vertex buffer** - buffer koji sadrži sve vertex podatke u nizu (vrhove, vektor normale, boje, kordinate tekstura i sl.)
2. **Index buffer** - isto kao i vertex buffer, sadrži niz vertex podataka, ali razlika je u tome što se podaci indeksiraju i time se odbacuje redudadnost vrhova te se ušteđuje na memoriji
3. **Vertex  array** - na njega se vežu vertex i index bufferi, te preko pokazivača koji razdvaja atribute (vrhove, boje …) unutar vertex/index buffera.
4. **Frame buffer** - buffer u kojem je zapisana generirana slika odnosno bitmapa
5. **Renderer buffer** - sadrže slike, ali je vezan za frame buffer
6. **Renderer** - klasa koja sadrži funkcionalnosti za renderiranje 
7. **Shader** - učitava GLSL kod shader, kompajlira i linka shadere u program
8. **Texture** - učitava slike i koristi je kao teksturu ili mapu

Primjer klase VertexBuffera: 

```cpp
#pragma once

class VertexBuffer
{
private:
	unsigned int m_RenderID;

public:
	VertexBuffer(const void* data,unsigned int size);
	~VertexBuffer();

	void Bind() const;
	void UnBind() const;
};
```

```cpp
#include "VertexBuffer.h"
#include "glad/glad.h"

VertexBuffer::VertexBuffer(const void* data,unsigned int size)
{
	glGenBuffers(1,&m_RenderID);
	glBindBuffer(GL_ARRAY_BUFFER, m_RenderID);
	glBufferData(GL_ARRAY_BUFFER, size, data, GL_STATIC_DRAW);
}

VertexBuffer::~VertexBuffer()
{
	glDeleteBuffers(1, &m_RenderID);
}

void VertexBuffer::Bind() const
{
	glBindBuffer(GL_ARRAY_BUFFER, m_RenderID);
}

void VertexBuffer::UnBind() const
{
	glBindBuffer(GL_ARRAY_BUFFER,0);
}
```

Primjer klase VertexArray:

```cpp
#pragma once
#include "VertexBuffer.h"

class VertexBufferLayout;

class VertexArray {
private:
	unsigned int m_RenderID;
public:
	VertexArray();
	~VertexArray();

	void Bind() const;
	void UnBind() const;

	void AddBuffer(const VertexBuffer& vb,const VertexBufferLayout& layout);
};
```

```cpp
#include "VertexArray.h"
#include "glad/glad.h"

#include "VertexBufferLayout.h"

VertexArray::VertexArray()
{
	glGenVertexArrays(1, &m_RenderID);
	glBindVertexArray(m_RenderID);
}

VertexArray::~VertexArray()
{
	glDeleteVertexArrays(1, &m_RenderID);
}

void VertexArray::Bind() const
{
	glBindVertexArray(m_RenderID);
}

void VertexArray::UnBind() const
{
	glBindVertexArray(0);
}   

void VertexArray::AddBuffer(const VertexBuffer& vb,const VertexBufferLayout& layout)
{
	Bind();
	vb.Bind();

	const auto& elements = layout.GetElements();
	unsigned int offset = 0;

	for (int i = 0; i < elements.size(); i++)
	{
		const auto& element = elements[i];
		glVertexAttribPointer(i, element.count, element.type, element.normalized,layout.GetStride(), (const void*)offset);
		glEnableVertexAttribArray(i);

		offset += element.count * element.GetSizeOfType(element.type);
	}
}
```

Kod potreban za renderiranje pravokutnika:

```cpp
// ...

int main()
{
	// ...
	
	// hard coded
	float planeVertices[] = { /*...*/ };

	VertexBuffer planeVertex(planeVertices, sizeof(planeVertices));
	
	VertexBufferLayout layoutPlane;
  layoutPlane.Push<float>(3); // vertex
  layoutPlane.Push<float>(3); // normal vector
  layoutPlane.Push<float>(2); // uv cordinates

	VertexArray planeVa;
  planeVa.AddBuffer(planeVertex, layoutPlane);
	
	Shader planeShader("res/shaders/PlaneVertex.glsl","res/shaders/PlaneFragment.glsl");
	
	//...

	while(/* Window is open */)
	{
		// ...

		render.Draw(planeVa,planeShader, 6);

		// ...
	}

	// ...
}
```

## Zaključak

Apstrakcija osnovnih funkcija OpenGL-a u klase predstavlja značajan napredak u smislu smanjenja linija koda, povećanja čitljivosti i olakšavanja razvoja aplikacija. Međutim, treba naglasiti da ova razina apstrakcije nije potpuno eliminirala sve OpenGL funkcije, već samo pružila jednostavnu apstrakciju za pojedine funkcije.

Potpuna apstrakcija OpenGL-a zahtijevala bi dublje razumijevanje njegovih internih mehanizama te bi rezultirala kompleksnijom strukturom klasa i funkcija. Iako ovo trenutno predstavlja jednostavnu apstrakciju, ostvareni napredak ukazuje na potencijalno poboljšanje kodiranja aplikacija koje koriste OpenGL te potrebu za daljnjim istraživanjem kako bi se postigla viša razina apstrakcije.