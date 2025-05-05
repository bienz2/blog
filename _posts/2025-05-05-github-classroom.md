---
layout: post
title: "GitHub Classroom Autograding -- Using CMake and Googletest"
categories: [teaching, grading, automation]
---

All of my upperclass/graduate classes use GitHub classroom autograding to provide students with automatic feedback on their work.  

Steps to creating a GitHub Classroom Assignment:
1. Register for an education account: https://github.com/education/teachers.  This gives you a number of GitHub Action hours for students to use (e.g. compiling and testing their code on GitHub Classroom).  
2. I highly recommend creating a second organization for all Github classroom content so that your normal account doesn't get flooded with student repositories.  I use ProfessorBienz for this https://github.com/ProfessorBienz.
3. Create a new public Github repository for your class.  Go to Settings and check that this is a `template repository`.  This is the repository that students will fork.  Here is my repository for Operating Systems: https://github.com/ProfessorBienz/OSHomework.
4. Create an additional public repository for all of your work.  Once a student forks a main repository, it is very difficult to get them any changes (for instance if you find a bug in some code or want to add an additional test).  However, if this second repository is a submodule within their forked repo and all updates are pushed to this submodule repository, they automatically get updates.  **This also stops them from editing test files to get around the autograder.  They cannot make any edits to the submodule.**
6. Create a new branch for both the main repo from step three corresponding to this homework
7. Add the submodule repo to the main repo with `git submodule add ...` if it does not already exist
8. Change your .gitmodules file to automatically checkout the correct branch of the submodule.  Here is an example:

```
[submodule "OSHomeworkSource"]
	path = OSHomeworkSource
	url = https://github.com/ProfessorBienz/OSHomeworkSource.git
        branch = homework1_src
```

9. Add your CMakeLists.txt.  I usually add the outermost one to the main repo and a secondary to the submodule repo.  However, you can put this all within the submodule if you want to prevent students from touching it.  Here is my outermost CMakeLists.txt:

```
# Sets Minimum Allowed CMake Version 
cmake_minimum_required(VERSION 2.8)

# Project Name 
project(homework)
set(SOURCE_NAME "OSHomeworkSource")

# Sets Language to C++, Uses C++11
enable_language(CXX)
set(CMAKE_CXX_STANDARD 11)

# Release Build : Compiles Optimized Code
set(CMAKE_BUILD_TYPE Release)

# Adds Compile Warnings
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wredundant-decls -Wcast-align -Wshadow")

# Will Include Files in Current Directory
set(hw_INCDIR ${CMAKE_CURRENT_SOURCE_DIR})
set(src_DIR ${CMAKE_CURRENT_SOURCE_DIR}/${SOURCE_NAME})

##################### 
## GOOGLETEST      ##
#####################
include(FetchContent)
if (CMAKE_VERSION VERSION_GREATER_EQUAL 3.5)
    set(GTEST_TAG 58d77fa8070e8cec2dc1ed015d66b454c8d78850)
else()
    set(GTEST_TAG e2239ee6043f73722e7aa812a459f54a28552929)
endif()
FetchContent_Declare(
  googletest
  GIT_REPOSITORY https://github.com/google/googletest.git
  # Specify the commit you depend on and update it regularly.
  GIT_TAG ${GTEST_TAG} 
)
# For Windows: Prevent overriding the parent project's compiler/linker settings
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
FetchContent_MakeAvailable(googletest)
enable_testing()
#####################

include_directories(${src_DIR})
include_directories(${hw_DIR})

# Add SRC Directory with Professor's Starter Code
add_subdirectory(${SOURCE_NAME})

# Create Library
# TODO : Include your new file here!
add_library(homework STATIC
    ${src_SOURCES}
    processes.cpp
    scheduler.cpp
    )

# Add Directory with Professor's Unit Tests
add_subdirectory(${SOURCE_NAME}/tests)

# Subdirectory with Example Programs
add_subdirectory(examples)
```

And within the submodule I have a second one for compiling files that students are not able to change:

```
set(CMAKE_INCLUDE_CURRENT_DIR ON)

set(src_SOURCES
    ${SOURCE_NAME}/src.hpp
    ${SOURCE_NAME}/src.cpp
    PARENT_SCOPE
    )
```


10. Add any template files to the outermost folder.  For instance, for Homework 1, I give students two files that they are to complete during the assignment:

```
#include "src.hpp"

// Fill in this method to complete Homework 1, Part 1
// Method is called by 'parent' process
void run_processes()
{

}
```

```
#include "src.hpp"

// Priority Scheduling 
//   -- Jobs with highest priority (lowest number) run first
//   -- If multiple jobs with same priority, lowest index runs first
void priority(int n_jobs, Job* jobs)
{
}

// Priority Scheduling with Round Robin 
//    -- Jobs with highest priority (lowest number) run first
//    -- If multiple jobs have same priority, run all in round robin 
//    -- Time slice for round robin passed as a parameter
void priority_rr(int n_jobs, Job* jobs, int time_slice)
{
}
```

11. Add all code that they cannot touch to the submodule repository.  I usually create two files, `src.cpp` and `src.hpp` containing all code that they can use but not edit.
12. Create unit tests for autograder.  I create a `tests` folder within the submodule and add all unit tests there.  **Make sure these are within the submodule so that students cannot edit them to automatically pass all tests.**
13. I like to provide by students with a `compile.sh` bash script that automatically pulls the latest submodule and compiles their code (it makes it easier for them to compile and also for classroom autograding in step 8).

14. Finally, you can add your assignments to GitHub Classroom https://classroom.github.com
15. Create a new classroom if you do not already have one
16. Create a new assignment, choose a deadline, and point it to your templated repository from step 3. 
17. **Make sure to select that the repository should be private so that students cannot see eachother's work.**
18. Click to copy only the default branch so that they do not have to worry about merging branches.
19. I recommend coming back to add unit tests later.  Sometimes Github Classroom crashes if you take too long adding unit tests and you have to start step 6 over.
20. You now have a new assignment.  Click on the link near that top that follows `Starter code from...`.  
    ![Starter Code](assets/github_classroom/starter_code.png)
21. You need to change the default branch of this repository:
        - Click Settings click the button with -> and <- to switch to a new branch.  Select the branch that corresponds to this homework.
    ![Switch Branch](assets/github_classroom/switch_branch.png)
23. Go back to your GitHub Classroom assignment.  Copy the provided link (also available at the top, to the left of `run tests`).  This is the link you should provide to your students.
    ![Assignment Link](assets/github_classroom/assignment_link.png)
25. I like to try the assignment out before giving it to the students.  If you paste the link you just copied, you can access the assignment.

26. Adding autograder tests: Click on 'edit' near the top of your github classroom homework.  You can add any tests you would like.  I usually create one test for 0 points that solely compiles their code, because it makes it more obvious to the students that their code did not compile if their is a GitHub Action labeled 'compile code' that fails.  I typically run `compile.sh` in this test.  You can then add one test that does `make test` to test all unit tests, however this will cause students to get either a 0 or 100.  Typically, I break up each unit test here and give a few points per test. 
