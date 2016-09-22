# TIQI - The ILIAS Question Importer

Create an ILIAS question pool from a text file (because doing this in ILIAS is annoying).

Write all your questions in a simple format in a text file that you can keep on your computer. Run this script and get an XML file with the questions that you can import into ILIAS as a "question pool" element from which you can create tests.


## Usage

### 1. Write your questions

Create a text file and write down your questions in the format detailed below.
You can use the example file `QuestionPool_test.txt` as an example and to test the script
(these are questions from my Semantic Web class, don't worry about the content).


### 2. Create an `xml` file that ILIAS can read

Call the TIQI script with:
    
    python tiqi.py <input text files>
    
For the example file `QuestionPool_test.txt`, call:

    python tiqi.py QuestionPool_test.txt

The output will be in `QuestionPool_test.txt.ilias.xml`.


### 3. Import the `xml` file in ILIAS

Inside some course, click on _Add new item_, chose _Question Pool Test_.
Create a new pool by chosing _Option 1: New Question Pool Test_ and give it a title and a description.

Inside this question pool element in the tab _Questions_ (should be the first one) you should see a list of questions which is empty at the moment.
On the upper right is a button _Import_, click that, select the `xml` file created with this script, click _Upload_, select all questions you want to import, click _Import_, ignore the error "No input type given. Set filename in constructor or choose setXMLContent()", go back to your course and you'll see the questions in your question pool.



## Format

A question has several parts:
 * the title
 * the question type, at the moment one of: 
 Multiple Choice Question (Single Answer) `[s]`,
 Multiple Choice Question (Multiple Answer) `[m]`,
 Cloze Question `[g]`
 * the text
 * several answer choices
 * awarded points per selected answer

Two questions are separated by an empty line.
The file ends with two empty lines.

Let's look at each question part in detail:

 * __Title and type__:
    The title and type are given in the first line of a question. This line has to be prefixed with `[t]` (for title) and _directly afterwards_ the type, either `[m]` (multiple choice several answers / checkboxes),  `[s]` (multiple choice single answers / radio buttons), or  `[g]` (cloze question / gaps to fill in). The title can only be one line, no longer. Don't put a space between the title and the type.

    __Warning__: If you forget `[t]` at the beginning of a line, no question is created. If you forget to follow up `[t]` with a type or use a non-recognized type, no question is created.


 * __Text__:
    The lines following the title are interpreted as being the question text. I do a bit of escaping of special characters to make this part valid XML/HTML, but you may need to extend this part or directly write HTML escape characters (the stuff with the &). Multiple whitespaces or tabs are ignored, but linebreaks are preserved. There doesn't have to be any text.

 * __Answers__:
    For multiple choice questions: The answer choices are listed after the question text. Each choice needs to start with either `_` (underscore, for correct answers) or `-` (minus, for incorrect answers). An answer can be several lines long. The same escaping is done for answer choices as for question text.
    
    For gap questions: `[gap]content of gap[/gap]` introduces a gap to be filled with the text inside the markup being the correct answer.

 * __Points__:
    The awarded points are calculated automatically. Each question is supposed to be worth 1 point. Partial points are distributed among the correct answers so that they give a total of 1 point. If there is no round way of doing that, the last answer choice get the remainder of points to get to a total of one which may be a bit more or less than the others. E.g., for 3 choices the first two will get 0.33 points, the last 0.34. For 6 choices the last will get 0.15 and the others 0.17. If you think this is too unfair, find a better way!

    For gap questions: Each gap is worth (1/number of gaps) and will give that amount of points if it is correctly filled, otherwise zero points. The default is case-sensitive answers, you can set this in the "itemmetadata" in the field "textgaprating" by changing the value "cs" to "ci".

    For multiple choice questions: If there is only one correct answer, this choice will receive 1 point if it is checked, the others receive 0 points if they are checked. If there are several correct choices, each correct choice receives (1/number of choices) points when checked and 0 otherwise. Incorrect choices receive (1/number of choices) points when UNchecked and 0 otherwise.

    __Warning__: If set multiple choices to be correct for a single-choice question, there will be more than one point assigned. This will write a warning, but doesn't seem to bother ILIAS, so the question is created with the points as you seemed to want it. Similarly, if there is no answer, zero points will be assigned, but ILIAS has no problem with that.


Examples:

    [t][m] Multiple Choice question with several answers: Encodings
    Which of the following are character encodings?
    - Unicode
    _ Latin-1
    _ UTF-8
    - Huffman coding
    _ UTF-16
    _ ASCII
    
    [t][s] Multiple Choice question with a single answer: Instance, Class or Property?
    You have the following RDFS statement:
       ex:hasFruit rdfs:domain ex:Plant .
    Is hasFruit an instance, a class or a property?
    _ Property
    - Class
    - Instance

    [t][g] Gap question: Classes and instances
    Santa Inc. is different from the individual Nico Claus.
    (ex:NicoClaus, ex:SantaInc)
    ex:SantaInc [gap]owl:differentFrom[/gap] ex:NicoClaus .
    
    

