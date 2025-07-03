https://github.com/MeatPounder11/taskmanager.cpp
#include <iostream>
#include <fstream>
#include <vector>
#include <string>
#include "json.hpp"

using json = nlohmann::json;

// Struttura di un task
struct Task {
    int id;
    std::string description;
    std::string status = "todo";
    std::string createdAt;
    std::string updatedAt;
};

// Funzione per ottenere data e ora attuali
std::string now() {
    time_t t = time(nullptr);
    char buf[100];
    strftime(buf, sizeof(buf), "%Y-%m-%d %H:%M:%S", localtime(&t));
    return std::string(buf);
}

// Legge tasks da file JSON
json loadTasks(const std::string& filename) {
    std::ifstream in(filename);
    if (!in) return json::array();

    try {
        json tasks;
        in >> tasks;
        if (!tasks.is_array()) return json::array();
        return tasks;
    } catch (json::parse_error& e) {
        std::cerr << "Errore di parsing JSON: " << e.what() << "\n";
        return json::array();
    }
}

// Salva tasks su file JSON
void saveTasks(const std::string& filename, const json& tasks) {
    std::ofstream out(filename);
    out << tasks.dump(4);
}

int main(int argc, char* argv[]) {
    if (argc < 2) {
        std::cerr << "Uso: ./taskmanager2 [add|list|update] [...]\n";
        return 1;
    }

    std::string comando = argv[1];
    std::string file = "tasks.json";
    json tasks = loadTasks(file);

    if (comando == "add") {
        if (argc < 3) {
            std::cerr << "Errore: manca la descrizione del task\n";
            return 1;
        }

        std::string descrizione;
        for (int i = 2; i < argc; ++i) {
            descrizione += argv[i];
            if (i < argc - 1) descrizione += " ";
        }

        int nuovo_id = tasks.empty() ? 1 : tasks.back().value("id", 0) + 1;
        json nuovo_task = {
            {"id", nuovo_id},
            {"description", descrizione},
            {"status", "todo"},
            {"createdAt", now()},
            {"updatedAt", now()}
        };

        tasks.push_back(nuovo_task);
        saveTasks(file, tasks);
        std::cout << " Task aggiunto con successo (ID: " << nuovo_id << ")\n";
    }

    else if (comando == "list") {
        if (tasks.empty()) {
            std::cout << " Nessun task presente.\n";
        } else {
            for (const auto& task : tasks) {
                std::cout << "[" << task["id"] << "] "
                          << task["description"] << " | "
                          << task["status"] << " | "
                          << task["createdAt"] << "\n";
            }
        }
    }

    else if (comando == "update") {
        if (argc < 4) {
            std::cerr << "Uso: ./taskmanager2 update <id> <nuovo_stato>\n";
            return 1;
        }

        int id_da_modificare = std::stoi(argv[2]);
        std::string nuovo_stato = argv[3];
        bool trovato = false;

        for (auto& task : tasks) {
            if (task.value("id", 0) == id_da_modificare) {
                task["status"] = nuovo_stato;
                task["updatedAt"] = now();
                trovato = true;
                break;
            }
        }

        if (trovato) {
            saveTasks(file, tasks);
            std::cout << " Stato del task " << id_da_modificare << " aggiornato a '" << nuovo_stato << "'\n";
        } else {
            std::cerr << " Task con ID " << id_da_modificare << " non trovato.\n";
        }
    }

    else {
        std::cerr << "Comando non riconosciuto: " << comando << "\n";
        return 1;
    }

    return 0;
}
