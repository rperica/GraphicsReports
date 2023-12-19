# Izvješće 6 - Kamera

Kamera u OpenGL-u određuje perspektivu i položaj gledanja unutar 3D scene. Za definiranje kamere, koristi se koncept pogleda (view) koji uključuje poziciju kamere, smjer gledanja, orijentaciju i perspektivu. Ove informacije se koriste za konstruiranje matrica transformacija koje se primjenjuju na objekte unutar scene.

LookUp matrica (View Matrix) je matrica transformacije koja opisuje položaj i orijentaciju kamere u sceni. Sastoji se od:

1. **Pozicija kamere** - definira se pozicija kamere u prostoru
2. **Smjer gledanja** - vektor koji određuje gdje kamera gleda
3. **Vektor desno** - smjer koji predstavlja pozitivnu x os u prostoru kamere
4. **Vektor gore** - smjer koji predstavlja pozitivnu y os u prostoru kamere

Za konstruiranje LookUp matrice dovoljni podaci su pozicija kamere, smjere gledanja i vektor gore. GLM biblioteka pruža funkciju *lookAt()* za konstruiranje view matrice. 

Deklaracija klase kamere:

```cpp
#pragma once
#include "glm/glm.hpp"
#include "glm/gtc/matrix_transform.hpp"

enum cameraMovement
{
	FORWARD,
	BACKWARD,
	LEFT,
	RIGHT
};

const float YAW = -90.0f;
const float PITCH = 0.0f;
const float ZOOM = 45.0f;
const float SPEED = 2.5f;
const float SENSIVITY = 0.1f;

class Camera
{
private:
	glm::vec3 m_Position;
	glm::vec3 m_Direction;
	glm::vec3 m_Right;
	glm::vec3 m_Up;
	glm::vec3 m_WorldUp;

	float m_yaw;
	float m_pitch;
	float m_speed;
	float m_sensivity;
	float m_zoom;

	void UpdateCamera();

public:
	Camera(glm::vec3 position= glm::vec3(0.0f, 0.0f, 0.0f),glm::vec3 worldUp=glm::vec3(0.0f,1.0f,0.0f),float yaw=YAW,float pitch=PITCH);
	~Camera();

	glm::mat4 View() const;
	
	void KeyboardInput(cameraMovement moveDirection,float deltaTime);
	void MouseProcess(float xoffset,float yoffset,bool constrainPitch=true);
	void ScrollProcess(float yoffset);

	inline const glm::vec3& GetPosition() const { return m_Position; }
	inline const glm::vec3& GetFront() const { return m_Direction; }
	inline const glm::vec3& GetUP() const { return m_WorldUp; }
	inline const float& GetZoom()const { return m_zoom; }
};
```

Da bi omogućili interaktivnost kamere, koristit ćemo callback funkcije. Callback funkcije u OpenGL-u se koriste za obradu korisnikovih ulaza kao što su pritisak tipki na tipkovnici, pokretanje miša ili drugi događaji vezani uz ulaz uređaja. Kroz callback funkcije, aplikacija može reagirati na korisnikove interakcije. 

Tri callback funkcije za kretanje kamere s WASD tipkama, zumiranje kamere i rotiranje kamere mišom. Funkcije su:

```cpp
// Kretanje kamere
void processInput(GLFWwindow* window)
{
    if (glfwGetKey(window, GLFW_KEY_ESCAPE) == GLFW_PRESS)
        glfwSetWindowShouldClose(window, true);

    if (glfwGetKey(window, GLFW_KEY_W) == GLFW_PRESS)
        camera.KeyboardInput(FORWARD, deltaTime);
    if (glfwGetKey(window, GLFW_KEY_S) == GLFW_PRESS)
        camera.KeyboardInput(BACKWARD, deltaTime);
    if (glfwGetKey(window, GLFW_KEY_A) == GLFW_PRESS)
        camera.KeyboardInput(LEFT, deltaTime);
    if (glfwGetKey(window, GLFW_KEY_D) == GLFW_PRESS)
        camera.KeyboardInput(RIGHT, deltaTime);
}

// Rotiranje kamere
void mouse_callback(GLFWwindow* window, double xpos, double ypos)
{
    if (firstMouse)
    {
        lastX = xpos;
        lastY = ypos;
        firstMouse = false;
    }
    
    float xoffset = xpos - lastX;
    float yoffset = lastY-ypos;
    lastX = xpos;
    lastY = ypos;
    
    camera.MouseProcess(xoffset, yoffset);
}

// Zumiranje kamere
void scroll_callback(GLFWwindow* window, double xoffset, double yoffset)
{
    camera.ScrollProcess(yoffset);
}
```

## Zaključak

Kamera u OpenGL-u je ključni element za definiranje perspektive i orijentacije gledanja unutar 3D scene. Kroz konstrukciju LookUp matrice, definira se položaj kamere i smjer gledanja. Callback funkcije se koriste za obradu korisničkih ulaza, omogućava nam da korisnik može upravljat kamerom. Ovakav sistem kamere se koristi u FPS igrama.