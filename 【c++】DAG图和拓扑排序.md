```
#include <iostream>
#include <string>
#include <vector>
#include <memory>
#include <queue>
#include <unordered_map>

class Digraph {
public:
  Digraph(const int cnt) {
    cnt_ = cnt;
    container_.resize(cnt);
    ingree_.resize(cnt);
  }

public:
  void AddEdge(const int from, const int to) {
    container_[from].push_back(to);
    ingree_[to]++;
  }

  void Dump() {
    for (int i = 0; i < container_.size(); i++) {
      std::cout << "Level: " << i << std::endl;

      for (auto &n : container_[i]) {
        std::cout << n << ", ";
      }

      std::cout << std::endl;
    }
  }
  
  std::vector<std::vector<int>> Travel() {
    std::unordered_map<int, bool> visited;
    std::vector<std::vector<std::vector<int>>> all_result(cnt_);


    // init the first level
    for (int i = 0; i < cnt_; i++) {
      std::vector<std::vector<int>> tr = travel_one(i, visited);
      for (int ii = 0; ii < tr.size(); ii++) {
        if (tr[ii].size() == 0) continue;

        std::cout << "Level: " << ii << ", size: " << tr[ii].size() << std::endl;
      }

      all_result[i] = tr;
    }

    std::vector<std::vector<int>> final_result(cnt_);

    for (int i = 0; i < cnt_; i++) {
      std::cout << "node: " << i << ", size: " << all_result[i].size() << std::endl;
      if (all_result[i].size() == 0) continue; // no any level

      for (int j = 0; j < all_result[i].size(); j++) {
        if (all_result[i][j].size() == 0) continue;

        for (auto &t : all_result[i][j]) {
          final_result[j].push_back(t);
        }
      }
    }

    return final_result;
  }

  std::vector<std::vector<int>> travel_one(const int c, std::unordered_map<int, bool>& visited) {
    std::queue<int> q;
    q.push(c);

    std::vector<std::vector<int>> result;

    while (!q.empty()) {
      std::queue<int> tmq;
      tmq.swap(q);

      std::vector<int> level;
      while (!tmq.empty()) {
        int cur = tmq.front();
        tmq.pop();

        if (visited[cur]) {
          continue;
        }
        visited[cur] = true;

        level.push_back(cur);

        for (int &o : container_[cur]) {
          if (visited[o]) {
            continue;
          }

          q.push(o);
        }
      } 

      result.push_back(level);
    }

    return result;
  }

private:
  int cnt_;
  std::vector<std::vector<int>> container_;
  std::vector<int> ingree_;
};

int main(int argc, char** argv) {

  Digraph d(7);
  d.AddEdge(0, 1);
  d.AddEdge(0, 2);
  d.AddEdge(1, 3);
  d.AddEdge(1, 4);
  d.AddEdge(4, 5);
  d.AddEdge(6, 6);

  // d.Dump();

  std::vector<std::vector<int>> results = d.Travel();

  for (int i = 0; i < results.size(); i++) {
    if (results[i].size() == 0) continue;

    std::cout << "Level: " << i << ", size: " << results[i].size() << std::endl;

    for (auto &n : results[i]) {
      std::cout << n << ", ";
    }

    std::cout << std::endl;
  }

  return 0;
}
```
