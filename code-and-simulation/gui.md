
install the desigegner
sudo apt update
sudo apt install qttools5-dev-tools qttools5-dev



In the .vscode, you should have a file called c_cpp_proporties.json. In the include path, add `/usr/include/x86_64-linux-gnu/qt5/**`and `/usr/include/x86_64-linux-gnu/qt6/`. This allows your intellisense to see QT headers. 