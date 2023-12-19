# Izvješće 3 - Shaderi i teksture

## Shaderi

Shaderi su programi koji se izvršavaju na grafičkoj kartici. Shaderi su programibilni dijelovi grafičkog cjevovoda. Postoje četri vrste shadera, a to su Vertex shader, Tessellation shader, Geometry shader i Fragment shader. Vertex i Fragment shader su obvezni u Modern OpenGL-u. Uloga Vertex shadera je da iz vertex buffera uzme vrhove te ih procesuira, a ostale atribute proslijedi dalje Fragment shaderu. Dok je uloga Fragment shadera da odredi boju svakom fragmentu. Jezik u kojem su napisani shaderu u OpenGL-u  je OpenGL Shading Language (GLSL). GLSL je jednostavni jezik sličan C-u.

Primjer Vertex shadera:

```glsl
#version 330 core
layout(location=0) in vec3 aPos;
layout(location=2) in vec2 aTexCor;

out vec2 TexCor;

uniform mat4 mvp;

void main()
{
	TexCor=aTexCor;
	gl_Position=mvp*vec4(aPos,1.0f);
}
```

Primjer Fragment shadera:

```glsl
#version 330 core
layout(location=0) out vec4 color;

in vec2 TexCor;

uniform sampler2D tex;
void main()
{
	color=vec4(texture(tex,TexCor).rgb,1.0f);
}
```

Svaki GLSL program trebao bi početi sa deklaracijom njegove verzije (npr. #version 330 core
za OpenGL 3.3). Nakon toga ide lista ulaznih varijabli sa ključnom riječi in, lista izlaznih
varijabli sa ključnom riječi out, te lista globalnih varijabli postavljenih preko CPU-a sa ključnom
riječi uniform. Sam kod za izvršavanje stavlja se u main funkciju, u kojoj se procesuiraju
spomenute varijable i postavlja vrijednost za izlazne varijable.

Shader klasa je abstraktirana klasa koja učitaj datoteke shadera, kompajlira ih i povezuje u program. Osim toga omogućuje postavljanje različitih tipova uniforma. Deklaracija shader klase:

```cpp
#pragma once
#include <iostream>
#include <string>
#include <sstream>
#include <fstream>

#include "glm/glm.hpp"

class Shader {
private:
	unsigned int m_RenderID;
	std::string m_VertexSource;
	std::string m_FragmentSource;
public:
	Shader(const std::string& vertexShader, const std::string& fragmentShader);
	~Shader();
	void CreateShader();
	unsigned int CompileShader(int type);
	void Bind() const;
	void Unbind() const;

	void SetUniform4x4(const std::string& name,const glm::mat4& value) const;
	void SetUniformVec3(const std::string& name, float x, float y, float z) const;
	void SetUniformVec3(const std::string& name, const glm::vec3& value) const;
	void SetUniformFloat(const std::string& name, const float& value) const;
	void SetUniformInt(const std::string& name, const int& value) const;

	inline const unsigned int GetID() const { return m_RenderID; }
};
```

## Teksture

Teksture su slike ili skupovi slika koje se koriste u računalnoj grafici kako bi se dodali detalji, boje ili uzorci površinama objekata. One su ključni element u stvaranju realističnih vizualnih efekata. 

Koriste se za:

- Dodavanje detalja objektima
- Definiranje boja i uzoraka
- Simuliranje materijala (mape)
- Dodavanje atmosfere (skybox)

Za učitvanje slika standardnih formata poput JPEG-a i PNG-a korištena je STB Image biblioteka. Tekstura ne mora biti već postojeća slika, nego se može i renderirati scena u teksturu, ovo se postiže tako da renderiramo u framebuffer. Ovo se koristi za kreiranje sjena, refleksija, post processinga i sl.

Deklaracija Texture klase:

```cpp
#pragma once
#include <string>
#include<vector>
#include "vendor/stb_image/stb_image.h"

class Texture
{
private:
	unsigned int m_RenderID;
	std::string m_FilePath;
	int m_Width, m_Height, m_BPP; 
	unsigned char* m_LocalBuffer;

public:
	Texture(const std::string& path);
	Texture();
	Texture(const int& width,const int& height);
	Texture(const std::vector<std::string>& faces);

	~Texture();

	void Bind(unsigned int slot=0) const;
	void UnBind() const;

	void BindCubeMap() const;
};
```

## Zaključak

Shaderi i teksture su ključni elementi u grafičkom programiranju. Shaderi kontroliraju izgled i ponašanje grafičkih elemenata, dok teksture dodaju detalje i teksture površinama. Proces učitavanja shadera i tekstura u OpenGL zahtijeva korake za pripremu i povezivanje podataka kako bi se uspješno koristili u vizualizaciji 3D scena.