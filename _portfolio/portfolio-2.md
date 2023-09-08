---
title: "Text-Based Game Development Framework"
excerpt: "Welcome to the Dynamic Text-Based Game Development Framework project, a multifaceted initiative designed to empower developers with sophisticated tools for creating engaging and interactive text-based games in C and C++. This project is distinguished by its emphasis on dynamic memory allocation, an expansive range of text manipulation algorithms, and a compelling gameplay experience.![houseprice](/images/evaluation.png){: .align-center width='350px'}"
collection: portfolio
---
<a id="top"></a>

***Guowang Zeng***

## Dynamic Category and Word Management (C)

At its core, this project introduces dynamic category and word management, revolutionizing the way textual content is handled. The `category_t` and `catarray_t` structures provide dynamic memory allocation capabilities, enabling the efficient storage and manipulation of categories and their associated words. The framework excels in handling memory allocation and deallocation gracefully, ensuring optimal performance while accommodating text expansion. Developers can effortlessly organize, expand, and modify the linguistic content of their games. These features are especially vital when dealing with unpredictable or evolving text data, giving creators the flexibility to adapt their narratives dynamically.

```c
#include <stdint.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>

struct country_tag{
    char name[64];
    uint64_t population;
};
typedef struct country_tag country_t;

country_t parseLine(char * line) {
  //WRITE ME
    if (line == NULL){ //check whether the input data is null or not
        fprintf(stderr, "The input data is missing, faild to process");
        exit(EXIT_FAILURE);
    }

    country_t ans;
    ans.name[0] = '\0';
    ans.population = 0;

    int index1 = 0, index2 = 0; //set initial index for looping the array

    char population_temp[64] = {""}; //set a temporary char type arrary for copying

    char * comma_position = strchr(line, ','); //find the index position of ','
    int comma_index = comma_position - line;

	for (int i=0; i<strlen(line); i++){ 
        if (index1 < comma_index){ //store country name which before ','
		    ans.name[index1] = line[i];
            ans.name[index1+1] = '\0';
		    index1++;
        }
        else { //store population into temp char which after ','
            population_temp[index2] = line[i+1];
            index2++;
        }
            
	}

    uint64_t n = atoi(population_temp); //convert char type to uint64_t type
    ans.population = n;

    printf("%s\n", ans.name);
    printf("%llu\n", ans.population);
 
  return ans;
}

void calcRunningAvg(unsigned * data, size_t n_days, double * avg) {
  //WRITE ME
    size_t avg_lenth = n_days - 6; //set the length of avg array

    if (data == NULL){ // check whether the input data is null or not
        fprintf(stderr, "The input data is missing, faild to process");
        exit(EXIT_FAILURE);
    }

    if (avg_lenth < 1){ // check whether the data is enough to compute
        fprintf(stderr, "The amount of data is less than 7, faild to compute");
        exit(EXIT_FAILURE);
    }

    if (avg_lenth >= 1){
        for (int i=0; i<avg_lenth; i++){ //choose which 7 days' domain to compute
            double accumulate = 0; //set the initial value for looping to add
            for (int j=0; j<7; j++){ //compute avg for 7 days' domain
                accumulate += data[j+i];
            }
            avg[i] = accumulate/7; //compute the avg
        }
    }
}

void calcCumulative(unsigned * data, size_t n_days, uint64_t pop, double * cum) {
  //WRITE ME
    if (data == NULL){ // check whether the input data is null or not
        fprintf(stderr, "The input data is missing, faild to process");
        exit(EXIT_FAILURE);
    }

    if (n_days < 1){ // check whether the data is enough to compute
        fprintf(stderr, "The amount of data is less than 1, faild to compute");
        exit(EXIT_FAILURE);
    }

    if (n_days >= 1){
        double initial = 0; //set initial value to accumulate
        for (int i=0; i<n_days; i++){ //compute the accmulate value
            initial += data[i];
            cum[i] = (100000 * initial)/pop;
        }
    }
}

void printCountryWithMax(country_t * countries,
                         size_t n_countries,
                         unsigned ** data,
                         size_t n_days) {
  //WRITE ME
    if (countries == NULL){ // check whether the input data is null or not
        fprintf(stderr, "The input data is missing, faild to process");
        exit(EXIT_FAILURE);
    }

    if (data == NULL){ // check whether the input data is null or not
        fprintf(stderr, "The input data is missing, faild to process");
        exit(EXIT_FAILURE);
    }

    char * country_name = NULL; //Initialize the variable
    unsigned number_cases = 0; //Initialize the variable

    for (int i=0; i<n_countries; i++){ // Find the largest value in n_countries‘ domain
        for (int j=0; j<n_days; j++){ // Find the largest value in n_days‘ domain
            if (number_cases < data[i][j]){ // Check if the value is bigger than the previous one
                number_cases = data[i][j]; // Update the number_cases when found the largest value
                country_name = countries[i].name; // Update the country name after finding the largest value

            }
        }
    }

    printf("%s has the most daily cases with %u\n", country_name, number_cases);
}
```

## Simple Text-Based Game Engine (C)

Our project offers a foundational text-based game engine implemented in C, empowering developers to construct interactive narratives with ease. This engine introduces two crucial classes: `Page` and `Story`. The `Page` class encapsulates essential page attributes such as type, number, text content, choices, and navigation logic. The `Story` class orchestrates the overall gameplay, ensuring story validity, managing the player's progression, and providing options to explore the story's depth or unveil an acyclic win path.

### Key Features

`Dynamic Memory Allocation:` The project showcases dynamic memory allocation in action, demonstrating how memory can be efficiently managed and freed during gameplay, preventing resource leaks and optimizing performance.
<br/><br/>
`Text Manipulation Algorithms:` A wide array of text manipulation algorithms, including string parsing, comparison, and dynamic array resizing, are showcased throughout the codebase, providing developers with valuable insights into these common techniques.
<br/><br/>
`Immersive Gameplay:` The text-based game engine offers an immersive gaming experience where players navigate through a branching storyline driven by their choices, culminating in either victory or defeat.
<br/><br/>
`Story Depth Analysis:` Developers can gain insights into the depth and structure of their narratives, facilitating debugging and refinement.
Acyclic Win Path Discovery: The framework aids developers in uncovering acyclic paths to victory, a critical aspect of creating engaging narratives.

```c
#include <stdint.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>

struct category_tag {
  char * name;
  char ** words;
  size_t n_words;
};
typedef struct category_tag category_t;

struct catarray_tag {
  category_t * arr;
  size_t n;
};
typedef struct catarray_tag catarray_t;

const char * chooseWord(char * category, catarray_t * cats);
void printWords(catarray_t * cats);

// when a word is seleceted from array.arr, we should revised the words in the array.arr to delete the reuse word. 
// arguments: 
// array: the return variable read from "storeCategory" function
// cate_word: the word randomly choose from "chooseWord" function
// reuse_index: a switch (only 0 and 1) to control whether reuse from array or not 
void revisedArray(catarray_t *array, const char *cate_word, int reuse_index) {
    // if reuse_index is 1, which is defalut situation same as step3
    if (reuse_index == 1)
        return;

    // else delete the word from the array which is used previouly, move the following words 1 position ahead and realloc new spaces
    else {
        char* cate_word_t = strdup(cate_word);
        for (int i = 0; i < array->n; i++) {
            for (int j = 0; j < array->arr[i].n_words; ++j) {
                if (strcmp(cate_word_t, array->arr[i].words[j]) == 0) {
                    --array->arr[i].n_words;
                    free(array->arr[i].words[j]);
                    for (int k = j; k < array->arr[i].n_words; k++) {
                        array->arr[i].words[k] = array->arr[i].words[k+1];  
                    }
                    array->arr[i].words = realloc(array->arr[i].words, array->arr[i].n_words*sizeof(*array->arr[i].words));
                }
            }
        }
        free(cate_word_t);
    }
}

// parse the template files line by line and for each line parse the characters which meet the conditions
// arguments: 
// file: the file provided in the one line argument 
// array: the return variable read from "storeCategory" function
// reuse_index: a switch (only 0 and 1) to control whether reuse from array or not 
void parseTemplate(char *file, catarray_t *array, int reuse_index) {
    FILE *f = fopen(file, "r");

    if (f == NULL) {
    perror("The file is faild to open, faild to process");
    exit(EXIT_FAILURE);
    }

    // initialize a string array to keep track of the words alrady been used and index for positioning
    char ** cate_word_copy = NULL;
    int used_index = 0;

    size_t sz = 0;
    char * line = NULL;

    while(getline(&line, &sz, f) >= 0) {
        char *modiefied_line = line;

        // searching for the first blank until the end of the line 
        for(int i=0; i< strlen(modiefied_line); i++) {
            // set index for whether excuting the category name or the integer
            int reset_index = 0;

            char * blank_pos1 = strchr(modiefied_line, '_');
            // if faild to find a first "_", then break 
            if (blank_pos1 == NULL) {
                break;
            }
            // find the second "_" for matching
            char * blank_pos2 = strchr(blank_pos1 + 1, '_');
            // if faild to find the second "_", then exit with error
            if (blank_pos2 == NULL) {
                    perror("Faild to find the '_' matched, faild to process");
                    exit(EXIT_FAILURE);
            }

            // print all the charaters before the "_" 
            int lenth = blank_pos1 - modiefied_line;
            for (int i=0; i<lenth; i++){ 
                 printf("%c",modiefied_line[i]);
            }

            // extract the category name which inside the two "_"
            int cate_len = blank_pos2 - blank_pos1 - 1;
            char *category = strndup(blank_pos1 + 1, cate_len);

            // set condition for step1
            if (array == NULL) {
                const char *cate_word = chooseWord("annimal", NULL);
                    printf("%s", cate_word);
            }
            
            // else continue excuting for step3 and step4
            else {
                for (size_t i = 0; i < array->n; i++) {
                    // check whether the array has the same name with the parsed category
                    if (strcmp(category, array->arr[i].name) == 0) {
                        // alter the index to 1 so we do not need to excute the condition for converted category name to integer
                        reset_index = 1;
                        const char *cate_word = chooseWord(category, array);
                        printf("%s", cate_word);
                        cate_word_copy = realloc(cate_word_copy, (used_index + 1)*sizeof(*cate_word_copy));
                        cate_word_copy[used_index] = strdup(cate_word); 
                        used_index++;
                        // excute "revisedArray" function when the reuse_index is 0
                        revisedArray(array, cate_word, reuse_index);
                    }
                }

                // convert string to integer
                int number = atoi(category);
                // if the category name is a integer which grater and equal than 1, then using back reference to print out the copy of words
                if (reset_index == 0 && number >= 1) {
                    // if the integer is greater than the previous stored accumnulated index, then exit with error
                    if (number > used_index) {
                        perror("No words are available for replacement, faild to process");
                        exit(EXIT_FAILURE);
                    }
                    cate_word_copy = realloc(cate_word_copy, (used_index + 1)*sizeof(*cate_word_copy));
                    printf("%s", cate_word_copy[used_index - number]);
                    cate_word_copy[used_index] = strdup(cate_word_copy[used_index - number]);
                    used_index++;
                }
                
                // exit with error when category name is a not valid integer or of at least one
                else if (reset_index == 0 && number < 1) {
                    free(line);
                    fclose(f);
                    perror("The category name is a not valid integer or of at least one");
                    exit(EXIT_FAILURE);
                }

            // reset index to be zero for next line parsing
            reset_index = 0;
            }

            // update the position for next blank searching
            modiefied_line = blank_pos2 + 1;
            free(category); 
        }

        // print out the rest of the line
        printf("%s", modiefied_line);
    }

    // free the stored array of copied category words
    for (int i=0; i<used_index; i++) {
        free(cate_word_copy[i]);
    }

    free(cate_word_copy); 
    free(line);
    fclose(f);
}

// create the array from words file
// arguments: 
// file: the file provided in the one line argument 
catarray_t *storeCategory(char *file) {

    FILE *f = fopen(file, "r");
    if (f == NULL) {
    perror("The file is faild to open, faild to process");
    exit(EXIT_FAILURE);
    }

    catarray_t *array = malloc(sizeof(*array));
    array->arr = NULL;
    array->n = 0;

    size_t sz = 0;
    char * line = NULL;

    while(getline(&line, &sz, f) >= 0) {
        char * colon_pos = strchr(line, ':');
        if (colon_pos == NULL) {
            perror("Faild to find the ':', faild to process");
            exit(EXIT_FAILURE);
        }
        
        // extract the category and word 
        int cate_len = colon_pos - line;
        char *category = strndup(line, cate_len);
        char *word = strdup(colon_pos+1);
        
        // remove the new line character "\n"
        for (int i =0; i < strlen(word); i++) {
            if (word[i] == '\n') {
                word[i] = '\0';
            }
        }

        // set index for not creating new catarray.arr if the category name already exist
        int reset_index = 0;

        // add a word in certain category when the category name is same
        for (int i = 0; i < array->n; i++) {
            if (strcmp(array->arr[i].name, category) == 0) {
                free(category);
                reset_index = 1;
                array->arr[i].words = realloc(array->arr[i].words, (array->arr[i].n_words + 1) * sizeof(*array->arr[i].words));
                array->arr[i].words[array->arr[i].n_words] = word;
                array->arr[i].n_words = array->arr[i].n_words + 1;
                break;
            }
        }
        
        // store the category name and word in a new category_t if is new
        if (reset_index == 0) {
            (*array).arr = realloc((*array).arr, ((*array).n + 1) * sizeof(*(*array).arr));
            (*array).arr[(*array).n].name = category;
            (*array).arr[(*array).n].words = malloc(sizeof(*(*array).arr[(*array).n].words));
            (*array).arr[(*array).n].words[0] = word;
            (*array).arr[(*array).n].n_words = 1;
            (*array).n++;
        }
    }

    free(line);
    fclose(f);
    return array;
}

// free the array point to the catarray and category
// arguments:
// array: the return variable read from "storeCategory" function
void freeCatArray(catarray_t *array) {
    if (array == NULL) {
        return;
    }

    for (size_t i = 0; i < array->n; ++i) {
        for (size_t j = 0; j < array->arr[i].n_words; ++j) {
            free(array->arr[i].words[j]);            
        }
        free(array->arr[i].name);
        free(array->arr[i].words);
    }
    free(array->arr);
    free(array);
}
```

## Advanced Text-Based Adventure Game (C++)

Taking a step further, the project presents an advanced text-based adventure game, implemented in C++. Building upon the principles established in the C-based engine, this game provides a richer and more complex gaming experience. The dynamic memory allocation prowess showcased in the previous components is further harnessed here, ensuring efficient memory management during dynamic gameplay.

### Key Features

`Dynamic Narrative Flow:` The C++ adventure game offers dynamic narrative flow where choices lead to divergent storylines, creating immersive and engaging experiences.
<br/><br/>
`Efficient Memory Handling:` Dynamic memory allocation is a core element, ensuring that memory is used optimally while accommodating variable text content.
<br/><br/>
`Interactive Storytelling:` Players embark on a captivating journey, making choices that significantly impact the storyline's direction. Winning Strategies: Developers and players can explore the game to discover multiple acyclic paths to victory, adding depth and replayability.
<br/><br/>
`Algorithmic Complexity:` Complex text manipulation algorithms for parsing, searching, and navigation are integrated into the gameplay, showcasing real-world applications of these essential coding techniques.

```c++
#include <algorithm>
#include <vector>
#include <cstdlib>
#include <iostream>
#include <cmath>
#include <string>
#include <fstream>
#include <sstream>
#include <set>
#include <queue>
#include <stack>

using namespace std;

// Page class to store page attributes and methods
class Page {
    public:
    string page_name;
    string page_type;
    int page_number;
    vector<string> text;
    vector<string> choices;
    vector<int> next_page;

    // constructor for Page class to initialize page attributes and methods 
    Page(string file) : page_name(file) {
        parseFile();
        parsePage();
    }
    // checkNumber method to check if the string is a number or not
    void checkNumber(string str){
        if (str[0] == '#') {
            return;
        }
        if (str[0] == '0'|| isdigit(str[0]) == false){
            cout << "Error: Invalid file format" << endl;
                exit (EXIT_FAILURE);
        }
        for (int i = 1; i < str.length(); i++){
            if (isdigit(str[i]) == false){
                cout << "Error: Invalid file format" << endl;
                exit (EXIT_FAILURE);
            }
        }
    }
    // parseFile method to parse the file and store the page number to page_number attribute
    void parseFile() {
        size_t dot_pos = page_name.find(".");
        size_t page_pos = page_name.find("page");

        if (dot_pos == string::npos) {
            cerr << "Error: Invalid file name" << endl;
            exit (EXIT_FAILURE);
        }

        if (page_pos == string::npos) {
            cerr << "Error: Invalid file name" << endl;
            exit (EXIT_FAILURE);
        }
        string str = page_name.substr(page_pos + 4, dot_pos - page_pos - 4);
        checkNumber(str);
        page_number = atoi(str.c_str());
    }
    // printPage method to print the page according to the page type respectively
    void printPage() {
        for (int i = 0; i < text.size(); i++) {
            cout << text[i] << endl;
        }
        if (page_type == "WIN") {
            cout << "\nCongratulations! You have won. Hooray!" << endl;
        } else if (page_type == "LOSE") {
            cout << "\nSorry, you have lost. Better luck next time!" << endl;
        } else {
            cout << "\nWhat would you like to do?\n" << endl;

            for (int i = 0; i < choices.size(); i++) {
                int colon_pos = choices[i].find(":");
                if (colon_pos == string::npos) {
                    cerr << "Error: Invalid file format" << endl;
                    exit (EXIT_FAILURE);
                }
                string str = choices[i].substr(0, colon_pos);
                checkNumber(str);

                next_page.push_back(atoi(str.c_str()));

                cout << " " << i + 1 << ". " << choices[i].substr(colon_pos + 1) << endl;
            }
        }
    }
    // parsePage method to parse the page and store the page type to page_type attribute 
    // and store the text and choices to text and choices attributes respectively  
    void parsePage() {
        // open the file and check wether the page file is NULL or not
        ifstream file(page_name);
        if (!file.is_open()) {
            cerr << "Error: File not found" << endl;
            exit (EXIT_FAILURE);
        }

        string line;
        getline(file, line);
        if (line == "LOSE" || line == "WIN") {
            page_type = line;

            string nextline;
            getline(file, nextline);
            if (nextline[0] != '#') {
                cerr << "Error: Invalid file format" << endl;
                exit (EXIT_FAILURE);
            }
            while (getline(file, line)) {
                text.push_back(line);
            }
        }

        else {
            int colon_pos = line.find(":");
            string str = line.substr(0, colon_pos);
            checkNumber(str);

            choices.push_back(line);
            page_type = "NEVIGATION";
           
            while (getline(file, line)) {

                choices.push_back(line);
                if (line[0] == '#') {
                    break;
                }   
            }

            while (getline(file, line)) {
                text.push_back(line);
            }

            choices.pop_back();              
        }
        // file.close();
    }
};

// Story class to store page attributes and methods
class Story {
    public:
    int current_page;
    int page_index;
    int win_index;
    int lose_index;
    int win_page_size;
    string directory_name;
    set<int> pages;
    set<int> win_pages;
    set<int> referenced_pages;
    vector<pair<int, int> > page_depth;
    // constructor for Story class to initialize story attributes and methods
    Story(string directory):current_page(1), page_index(0),
    win_index(0), lose_index(0), directory_name(directory) {
        checkStory();
    }
    // checkStory method to check the story is valid or not 
    // and check each page can be reached or not by the page choices and vice versa
    // and check the story has at least a win page/lose page or not
    void checkStory() {
        
        for (int i = 1;; i++) {
            stringstream ss;
            ss << directory_name << "/page" << i << ".txt";
            ifstream file((ss.str()).c_str());
            if (!file.is_open()) {
                break;
            }
            // increment the page size and store the page number to pages set
            ++page_index;
            pages.insert(i);

            Page page_name = Page(ss.str());
            if (page_name.page_type == "WIN") {
                win_index = 1;
                // increment the win page size and store the win page number to win_pages set
                win_pages.insert(i);
            } else if (page_name.page_type == "LOSE") {
                lose_index = 1;
            }

            for (int i = 0; i < page_name.choices.size(); i++) {
                int colon_pos = page_name.choices[i].find(":");
                if (colon_pos == string::npos) {
                    cerr << "Error: Invalid file format" << endl;
                    exit (EXIT_FAILURE);
                }

                string str = page_name.choices[i].substr(0, colon_pos);
                page_name.checkNumber(str);
                // store the refernced page number to referenced_pages set
                referenced_pages.insert(atoi(str.c_str()));

            }

        }
     
        if (win_index != 1 || lose_index != 1) {
            cerr << "Error: Invalid story file" << endl;
            exit (EXIT_FAILURE);
        }
        if (referenced_pages.size() > page_index) {
            cerr << "Error: Invalid story file" << endl;
            exit (EXIT_FAILURE);
        }
        for (int i = 2; i <= page_index; i++) {
            if (referenced_pages.find(i) == referenced_pages.end()) {
                cerr << "Error: Invalid story file" << endl;
                exit (EXIT_FAILURE);
            }
        }
        // store the win pages size
        win_page_size = win_pages.size();
    }
    // startStory method to start the story and print the first page of the story 
    // and get the user input and check the user input is valid or not
    // and print the next page according to the user input
    void startStory () {

        while (current_page != 0) {
            stringstream ss;
            ss << directory_name << "/page" << current_page << ".txt";
            Page page_name = Page(ss.str());

            page_name.printPage();

            if (page_name.page_type == "WIN") {
                current_page = 0;
                exit (EXIT_SUCCESS);
            } else if (page_name.page_type == "LOSE") {
                current_page = 0;
                exit (EXIT_SUCCESS);
            } else {
                while (true) {
                    int repeat_index = 0;
                    string choice;
                    cin >> choice;
                
                    vector<string> choices_number;

                    for (int i = 0; i < page_name.choices.size(); i++) {
                        //convert i to string
                        stringstream ss;
                        ss << i+1;
                        choices_number.push_back(ss.str());
                    }

                    if (find(choices_number.begin(), choices_number.end(), choice) == choices_number.end()) {
                        cerr << "That is not a valid choice, please try again" << endl;
                        repeat_index = 1;   
                    }

                    if (repeat_index == 0) {
                        current_page = page_name.next_page[atoi(choice.c_str()) - 1];
                        break;
                    }
                }
            }
        }
    }
    // printStoryDepth method to print the story depth of each page in the story by using BFS with queue
    void printStoryDepth () {

        queue<int> page_queue;

        page_depth.push_back(make_pair(1, 0));
        page_queue.push(1);
           
        while (!page_queue.empty()) {

            current_page = page_queue.front();
            page_queue.pop();

            stringstream ss;
            ss << directory_name << "/page" << current_page << ".txt";
            ifstream file((ss.str()).c_str());
            if (!file.is_open()) {
                break;
            }

            Page page_name = Page(ss.str());

            for (int i = 0; i < page_name.choices.size(); i++) {
                int colon_pos = page_name.choices[i].find(":");
                if (colon_pos == string::npos) {
                    cerr << "Error: Invalid file format" << endl;
                    exit (EXIT_FAILURE);
                }

                string str = page_name.choices[i].substr(0, colon_pos);
                page_name.checkNumber(str);

                page_queue.push(atoi(str.c_str()));
                // store the page depth in page_depth vector
                for (int j = 0; j < page_depth.size(); j++) {
                    if (page_depth[j].first == current_page) {
                        page_depth.push_back(make_pair(atoi(str.c_str()), page_depth[j].second + 1));
                    }
                }

                for (int k = 0; k < page_depth.size() - 1; k++) {
                    if (page_depth[k].first == atoi(str.c_str())) {
                        page_depth.pop_back();
                    }
                }
            }
            
        }

        //sort the page_depth vector with respect to the first element
        sort(page_depth.begin(), page_depth.end());
        
        // print the page_depth with respect to each page
        for (int i = 0; i < page_depth.size(); i++) {

            cout << "Page " << page_depth[i].first << ":" << page_depth[i].second << endl;
            
            if (pages.find(page_depth[i].first) != pages.end()) {
                pages.erase(page_depth[i].first);
            }
        }

        // print the unreachable pages;
        for (set<int>::iterator it = pages.begin(); it != pages.end(); ++it) {
            cout << "Page " << *it << " is not reachable" << endl;
        }
    }
    // printAcyclicWinpath method to initialize the visited/path set/vector and call the acyclicWinpath method
    void printAcyclicWinpath () {

        set<int> visited;
        vector<int> path;

        recursiveStory(path, visited, 1);

        //print out unwinaable pages
        if (win_pages.size() == win_page_size) {
            cout << "This story is unwinnable!" << endl;
        }
    }
    // recursiveStory method to print the acyclic win path of the story by using recursive DFS
    void recursiveStory (vector<int> path, set<int> visited, int choice_number) {
     
        path.push_back(choice_number);
        visited.insert(choice_number);

        stringstream ss;
        ss << directory_name << "/page" << choice_number << ".txt";
        Page page_name = Page(ss.str());

        if (page_name.page_type == "WIN") {

            win_pages.erase(choice_number);
            // print path vector with respect to each page in the path vector 
            for (int i = 0; i < path.size(); i++) {
                if(i == path.size() - 1){
                    cout << path[i] << "(win)" << endl;
                }
                else{
                    stringstream ss;
                    ss << directory_name << "/page" << path[i] << ".txt";
                    Page page_name = Page(ss.str());

                    int choice;

                    for (size_t j = 0; j < page_name.choices.size(); j++) {

                        int colon_pos = page_name.choices[j].find(":");
                        string str = page_name.choices[j].substr(0, colon_pos);
                        int page_number = atoi(str.c_str());

                        if (page_number == path[i+1]) {
                            choice = j + 1;
                        }
                    
                    }

                cout << path[i] << "("<< choice << ")" << ",";

                }
            }   
        } 
        else {
            for (int i = 0; i < page_name.choices.size(); i++) {

                int colon_pos = page_name.choices[i].find(":");

                if (colon_pos == string::npos) {
                    cerr << "Error: Invalid file format" << endl;
                    exit (EXIT_FAILURE);
                }

                string str = page_name.choices[i].substr(0, colon_pos);
                page_name.checkNumber(str);
                // if the page is not visited win or lose page, call the recursiveStory method
                if (visited.find(atoi(str.c_str())) == visited.end()) {
                    recursiveStory(path, visited, atoi(str.c_str()));
                }
            }
        }  
    }  
};


// step1
// int main(int argc, char ** argv) {
//     if (argc != 2) {
//     perror("The number of input argument is wrong, faild to process");
//     exit(EXIT_FAILURE);
//     }

//     Page page(argv[1]);
//     page.printPage();

//     return EXIT_SUCCESS;
// }

// step2
// int main(int argc, char ** argv) {
//     if (argc != 2) {
//     perror("The number of input argument is wrong, faild to process");
//     exit(EXIT_FAILURE);
//     }

//     Story story(argv[1]);
//     story.startStory();

//     return EXIT_SUCCESS;
// }

// step3
// int main(int argc, char ** argv) {
//     if (argc != 2) {
//     perror("The number of input argument is wrong, faild to process");
//     exit(EXIT_FAILURE);
//     }

//     Story story(argv[1]);
//     story.printStoryDepth();

//     return EXIT_SUCCESS;
// }

// step4
//int main(int argc, char ** argv) {
//    if (argc != 2) {
//    perror("The number of input argument is wrong, faild to process");
//   exit(EXIT_FAILURE);
//    }

//    Story story(argv[1]);
//    story.printAcyclicWinpath();

//    return EXIT_SUCCESS;
// }
```

## Conclusion

> The Dynamic Text-Based Game Development Framework is a comprehensive and educational resource for developers interested in text-based game development and dynamic memory allocation. It emphasizes efficient memory management, a wide array of text manipulation algorithms, and immersive gameplay experiences. By exploring this project, developers can uncover the dynamic nature of memory allocation and gain valuable insights into algorithms commonly used in text processing. Furthermore, they can harness these skills to craft captivating and interactive narratives that offer players unique and engaging experiences.

[<center>↑ BACK TO TOP ↑</center>](#top)

  <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
  <script>
    $(document).ready(function() {
      $('a[href="#top"]').click(function() {
        $('html, body').animate({ scrollTop: 0 }, 'slow');
        return false;
      });
    });
  </script>
