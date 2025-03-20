```cpp
#pragma once

#include "shader.h"
#include <glm/glm.hpp>
#include <vector>

struct vertex_t {
    glm::vec3 pos;
    glm::vec3 norm;
    glm::vec2 tex_coords;
};
static_assert(sizeof(vertex_t) == 8 * sizeof(float));

enum class texture_type_e {
    diffuse,
    specular,
};

struct texture_t {
    GLuint id;
    texture_type_e type;
};

class mesh_t {
  public:
    mesh_t(const std::vector<vertex_t> &vertices,
           const std::vector<GLuint> &indices,
           const std::vector<texture_t> &textures) : m_vertices{vertices}, m_indices{indices}, m_textures{textures} {
        glGenVertexArrays(1, &m_vao);
        glBindVertexArray(m_vao);

        std::cout << "mesh vertices count : " << m_vertices.size() << " , indices count : " << m_indices.size() << " , texture count " << m_textures.size() << "\n";

        GLuint vbo;
        glGenBuffers(1, &vbo);
        glBindBuffer(GL_ARRAY_BUFFER, vbo);
        glBufferData(GL_ARRAY_BUFFER, m_vertices.size() * sizeof(vertex_t), m_vertices.data(), GL_STATIC_DRAW);

        GLuint ebo;
        glGenBuffers(1, &ebo);
        glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, ebo);
        glBufferData(GL_ELEMENT_ARRAY_BUFFER, m_indices.size() * sizeof(GLuint), m_indices.data(), GL_STATIC_DRAW);

        glEnableVertexAttribArray(0);
        glVertexAttribPointer(0, 3, GL_FLOAT, false, sizeof(vertex_t), (void *)0);
        glEnableVertexAttribArray(1);
        glVertexAttribPointer(1, 3, GL_FLOAT, false, sizeof(vertex_t), (void *)(3 * sizeof(float)));
        glEnableVertexAttribArray(2);
        glVertexAttribPointer(2, 2, GL_FLOAT, false, sizeof(vertex_t), (void *)(6 * sizeof(float)));

        glBindVertexArray(0);
    }

    auto draw(const shader_t &shader) -> void {
        auto diffuse_tex_n = 1, specular_tex_n = 1;
        for (auto i = 0; i < m_textures.size(); ++i) {
            glActiveTexture(GL_TEXTURE0 + i);
            glBindTexture(GL_TEXTURE_2D, m_textures[i].id);

            switch (m_textures[i].type) {
            case texture_type_e::diffuse:
                shader.set_uniform("texture_diffuse_" + std::to_string(diffuse_tex_n++), i);
                break;
            case texture_type_e::specular:
                shader.set_uniform("texture_specular_" + std::to_string(specular_tex_n++), i);
                break;
            }
        }
        glActiveTexture(GL_TEXTURE0);

        glBindVertexArray(m_vao);
        glDrawElements(GL_TRIANGLES, m_indices.size(), GL_UNSIGNED_INT, 0);
        glBindVertexArray(0);
    }

  private:
    std::vector<vertex_t> m_vertices;
    std::vector<GLuint> m_indices;
    std::vector<texture_t> m_textures;
    GLuint m_vao;
};
```


```cpp
#pragma once

#include "assimp/Importer.hpp"
#include "assimp/postprocess.h"
#include "assimp/scene.h"
#include "mesh.h"

class model_t {
  public:
    auto load(std::string_view path) -> void {
        auto importer = Assimp::Importer{};
        auto scene = importer.ReadFile(path.data(), aiProcess_Triangulate | aiProcess_GenSmoothNormals | aiProcess_FlipUVs | aiProcess_CalcTangentSpace);
        if (!scene || scene->mFlags & AI_SCENE_FLAGS_INCOMPLETE || !scene->mRootNode) {
            std::cerr << importer.GetErrorString() << "\n";
            exit(-1);
        }
        m_directory = path.substr(0, path.find_last_of("/"));
        process_node(scene->mRootNode, scene);
    }

    auto draw(const shader_t &shader) -> void {
        for (auto &mesh : m_meshes) {
            mesh.draw(shader);
        }
    }

  private:
    auto process_node(const aiNode *node, const aiScene *scene) -> void {
        for (auto i = 0; i < node->mNumMeshes; ++i) {
            process_mesh(scene->mMeshes[node->mMeshes[i]], scene);
        }
        for (auto i = 0; i < node->mNumChildren; ++i) {
            process_node(node->mChildren[i], scene);
        }
    }

    auto process_mesh(const aiMesh *mesh, const aiScene *scene) -> void {
        auto vertices = std::vector<vertex_t>{};
        auto indices = std::vector<GLuint>{};
        auto textures = std::vector<texture_t>{};

        for (auto i = 0; i < mesh->mNumVertices; ++i) {
            // 一个顶点可能有多个贴图通道，这里只加载一个通道
            vertices.emplace_back(vertex_t{glm::vec3{mesh->mVertices[i].x, mesh->mVertices[i].y, mesh->mVertices[i].z},
                                           glm::vec3{mesh->mNormals[i].x, mesh->mNormals[i].y, mesh->mNormals[i].z},
                                           mesh->mTextureCoords[0] ? glm::vec2{mesh->mTextureCoords[0][i].x, mesh->mTextureCoords[0][i].y} : glm::vec2{0, 0}});
        }

        for (auto i = 0; i < mesh->mNumFaces; ++i) {
            for (auto j = 0; j < mesh->mFaces[i].mNumIndices; ++j) {
                indices.emplace_back(mesh->mFaces[i].mIndices[j]);
            }
        }

        load_texture(scene->mMaterials[mesh->mMaterialIndex], aiTextureType_DIFFUSE, textures);
        load_texture(scene->mMaterials[mesh->mMaterialIndex], aiTextureType_SPECULAR, textures);

        m_meshes.emplace_back(vertices, indices, textures);
    }

    auto load_texture(const aiMaterial *material, aiTextureType type, std::vector<texture_t> &textures) -> void {
        for (auto i = 0; i < material->GetTextureCount(type); ++i) {
            auto s = aiString{};
            material->GetTexture(type, i, &s);

            textures.push_back(texture_t{
                .id = shader_t::load_texture(m_directory + "/" + s.C_Str()),
                .type = type == aiTextureType_DIFFUSE ? texture_type_e::diffuse : texture_type_e::specular});
        }
    }

  private:
    std::vector<mesh_t> m_meshes;
    std::string m_directory;
};
```