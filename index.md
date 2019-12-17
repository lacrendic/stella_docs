---
layout: page
---

# University student retention pre-process

The idea of the pre-process is to develop a statistical model that allows us to identify those students which present a high risk of dropping out of university. The statistical model is of course just one small aspect of the whole process. Hence, it is important that not only the IT department be involved. They are a necessary actor, since they retrieve the data that the model needs from the university systems. But it is also crucial that the department in charge of student retention (DSR from now on) be involved: otherwise, there will be no way to target the students identified by the model. So this has to be insisted on when beginning the process.

There is an important caveat to take into consideration. The current statistical models used by Stella do not have a temporal component. They just see a picture of the student: that is, the same student, in different periods, is a different student in the eyes of the model. That is a huge TODO, because there is obviously very valuable cross-information that we are currently missing. I suppose the model could be hacked into being multi-period by including the temporal component as duplicate columns. But that's just ugly, and unnecessary, since time series *are* a thing. In any case, this has to be considered when discussing with the university which data they have. For example, if they have some information only for some periods, it may be not so useful for the model.

There is another important thing to take into consideration, which can help decide the direction and rhythm that we give the project. Some universities (such as UdL) will be mostly interested in results. Prediction results. Models. Even to the point where they will question the need for the DSR to be a part of the project. Those types of clients will want to get to the last stage (the completed model) as soon as possible. In those cases, it is our job to slow things down a little bit, involve the DSR, and make sure they are not just skimming the checkpoints but actually checking them. But also, in those cases, we can move faster through the process. On the other hand, there will be clients that are less impatient for results and for which the process of analyzing the data will be almost more interesting than the final model. Because, in general, they don't really know their data. They could answers questions like *What percentage of your students are foreign?* but not something deeper like *How many students that do badly in the first exam of the period go on to drop out?*. So they will be very interested in all the different reports and charts that we generate, and probably there will be more time to analyze the data, produce reports, and give them feedback on how to use their data better. 

In that regard, if we get enough experience (and I think Fran is already at that point), Stella could become a more comprehensive consulting experience, rather than just a drop out prediction model + an app for handling retention strategies. I know this transformation of Stella is already an objective but what I want to point out is that, in some ways, that transformation is already a partial reality in some projects. There is valuable know-how and expertise in previous Stella projects. It would need some standardizing and systematizing, but it is there.

**As a preface**: I will describe everything *as done nowadays*. Many of the insights are universal, but some provisions detailed here could be superseded by improving / redesigning parts of the process. I will be including notes and suggestions to that end throughout the document.

## Stage I: getting the data

The pre-process starts with an initial conversation where the university representative (probably from IT) details which data they can provide on students. There is some data that is usually available and valuable for the model so you should always try to get it:

 - **Grades**: this is usually of max importance for the model. Later we will give more details as to which data it usually includes and how to use it for the model. Same for the other categories we now list.
 - **Assistance**: some form of data indicating how consistent the students are in attending classes. For example, how many classes they have attended in the current period, as a % of the total.
 - **Financial info**: financial relationship between the student and the university. How much a student owes the university in a given period, for example, or how many days is he late for each of the periodic payments he must make to the university.
 - **Library**: library activity can be of some value. For example, students who borrow a lot of books tend to drop out less.
 - **Register phase**: there is usually some data indicating how long the students take to register for the new period. If they are quick to register, it generally means they are less prone to dropping out.
 - **Surveys**: universities routinely conduct surveys among students. Some of that data can be useful for the model (for example, data related to computer and cell phone use).

Something to consider: it is possible that the university has information dating many years into the past. Since universities are always changing, it is usually not worth it to look too far back: getting the data of the last 5 years for example is reasonable, but data from 15 years ago will likely not be as useful. Now, it is important to always consider how many students are we including with the data we have discussed with the university. Are there any students that are maybe being left out of the model? A group for which the model will maybe not work so well? 

One important group, that usually represents a big part of the drop out, are *new students*. Also, since they are new, there are some indicators that can't be constructed for them (for example, their cumulative GPA, their cumulative debt with the university, or their answers to internal surveys). And if we consider the future possibility of adding a temporal component to the model, relatively there will be even less information available to predict drop out risk for new students. This results in two considerations. First, there will be **two different models**: one for new students, another for continuity students. Second, there is some extra information that we will try to get from the university, that will be especially valuable for new students:

- **Personal info**: there is usually data of the students just before they entered university, which can provide useful information. Sex, nationality, region of origin in the country, high school GPA, age, English level. 
- **Admission info**: data related to the application of the student to his particular university. Reason why they chose their career or university, results in some kind of admission exam for the university or in some psychological evaluation.

When sensitive information (psychological results for example) is being discussed, the university may be reticent to provide it, but it is important to insist. One way to convince them is to say something along the following lines: *you can anonymize the data for now, for example masking the real names of the fields, but provide it anyway, and then we can see if the model thinks the data is important*. 

## Stage II: drop out mark

Once the data of interest has been agreed upon, the university has to send it. I guess this is usually done in the form of spreadsheets. Now, the objective of this stage and the next is to get to the following state, so we can apply the R scripts that already exist: a state where all the data that the university sent is consolidated in only one spreadsheet, where the columns that are not useful for the model are not included. Where each row in the spreadsheet corresponds to a *unique student* (we clarify this concept below), and each row indicates whether the student dropped out in the given period. In conclusion: one spreadsheet to rule them all.

There are a few things we need to achieve with the data at this stage, so we can produce the consolidated spreadsheet. We will explain them now in detail.

### Student consolidation

The university sends several spreadsheets, all of which have information on the students. We need to decide on a couple of things:

- First, which students will be considered for the model. To that end, usually one of the spreadsheets sent by the university will be deemed the **base file**. We will take that as our main file, and then join the useful information from the other spreadsheets in the form of new columns. 
- Second, how to identify the same student across different spreadsheets, so the consolidation between them can be made.

Now, an important problem is what to do about *duplicated students* in any given file (that is, different rows with different information, but the same value in the field identifying the student). We only want to keep records for *unique students*. So a decision has to be made as to what constitutes a unique student. Since the model is not time-conscious, usually we consider a pair `(student id, period)` as a unique record, and that is how most university data is organized (grades or financial information, for example, obviously include the time component). 

Even so, there may be some duplicate records (e.g. a student that is registered in two different careers at the same time). Usually, those cases are few, so the duplicates can be eliminated. All of this has to be done together with the university though, so they understand and approve of the decisions being made.

There is an issue that can produce duplicate students and other problems: manually entered data. For example, at UdL there was a survey about digital media use, where the student id field was entered by the student, and there were clearly some who used the wrong id (and one that even wrote her name). So when asking the university for data, it is important to **know their origin and how the data was obtained**. In cases such as this one, where there are typos, one has to explore the data. If it looks mostly OK, it can be used after cleaning it up (e.g. remove duplicates caused by typos). It it looks too messy, maybe the file should not be used at all (a decision that should be taken together with the client).

Another possible difficulty in homologating students across different files is the time description. Since the unique record is usually identified as `(student id, period)`, we need to be able to identify the same student and period in different spreadsheets. The student component is usually not a problem: the same student id is used in all different files (although some cleaning may be necessary, as already described). The time component can be more tricky, and at least two things have to be considered:

- Some files may not have a periodic component. For example, admission data or some surveys may be taken just once per student, rather than each period. In those cases, we join the valuable information for a given `student id` with all instances `(student id, period)` present in the base file. That is, for the same `student id`, we will have the same information (of a survey, for example), duplicated across different periods.
- The time component in different files may take different forms. In some, the time could be identified as a date, like `2019-08-18`, or as a code, like `2019-2`, or as a code in a different format, like `20192`. The varying forms should be homologated as an early step in the process. For this, the university should describe the periods in terms of date ranges: e.g. `2019-2` goes from August 2019 to January 2020. Also, it is important to remember that that description could depend on other fields, like the period regime of the career: e.g. `2019-2` is different for a biannual structure and for a quarterly structure.

### Drop out mark

The drop out mark (*marca de deserción*) indicates whether a student dropped out in a given period. Very often, universities do not have this information readily available, so it has to be calculated as part of the process. So for every *unique student* (as described above) there should be a column in the consolidated spreadsheet that indicates whether the student is a desertor or not. This part should be done with a lot of input from the university, for a few reasons:

- The criteria for considering someone a desertor should be clearly defined. This may involve cross-reference between different spreadsheets / data origins.
- There may be exceptions to be made for the rule. For example, as it was at UdL, there may be students that finish all their credits, but are not officially graduated for some reason (e.g. some pending procedure or exam that they never complete). So they could seem as desertors: they never graduated yet they are not currently enrolled. Students in that type of situation can constitute a problem for the client, but it is a separate phenomenon and should probably not be considered as desertion. 
- It may be necessary to ask for further information. For example, apart from all the spreadsheets that could be valuable for the model, at UdL we used another spreadsheet containing information about **Curricular progress**.

After clarifying all this with the university, the drop out mark has to be calculated (as a boolean value) and appended to the base file as a new column. After that, a report has to be constructed, so the client can look at the results and validate them. An example of such a report for UdL can be found here:

https://bit.ly/2Pqwaoj

We will give some explanation here anyway. The report is intended as a way for the client to check our calculation of the drop out mark, so it contains some general indicators that should look reasonable to them. At the most basic level, we can say something like *according to our calculations, 95% of your students drop out* to which they could (rightfully) reply *that sounds way too high, please check your calculations*, or *maybe we need to be sure we defined the drop out criteria correctly*.

As a quick heuristic, here are some referential guidelines for our internal checking (before consulting with the university):

- Drop out should be higher in the first periods of a career. For example, there is usually a higher drop out rate among first year students than older students.
- Drop out rates should be approximately cyclical. Supose a biannual regime. If in 2017 75% of the drop out was in the first semester, and 25% in the second, then those numbers should be similar for 2018.
- For the first period of a regime (first year of an annual regime, first semester of a biannual regime, etc.), a 50% drop out rate is kind of high but not unheard-of. Much higher than that (higher than 60% maybe) is too high and suspicious.

### Drop out report

As we said before, this is usually the first important checkpoint with the client: we deliver an artifact (a report of the kind we linked above), discuss it and validate it with the client, and finally get the OK to procceed with the next stage. The report has several charts / plots of interest, and the ones we explain here (or the ones included in the linked report) are not an exhaustive list at all: much more can be done.


**Notes**: there are some TODOs relative to this stage which could be very helpful and save a lot of time. I will mention them superficially now, but it is important at some point to sit down, think about them carefully, and give concrete proposals to improve the process. In particular, the Stella web app should play a much bigger role in the new and improved process: as it stands, it is used only at a late stage, when the model is already completed.

- **From spreadsheets to RDB**: there is a lot of time wasted in cleaning and consolidating the data that the university sends in the form of spreadsheets. To some degree, that is unavoidable, since all IT systems have their errors and any data science project will inevitably involve data wrangling. But it would be a great help if, instead of just sending a bunch of spreadsheets, Stella clients had to upload the data to an app, like it is the case for DarwinEd. For example, this could allow us to define a student model in a RDB, which will save us the work of identifying the same student across different data origins.
- **Drop out mark**: I don't think it is too crazy to let the universities handle this part, unless there are important reasons to think they won't be able to do it right. Calculating the drop out mark takes time. There is something to consider though: in the process of calculating the drop out mark, there is a back and forth with the client where a lot is clarified about the university (e.g. whether the periods are annual, biannual, mixed, etc.). It is important to not lose that info in case this task is given to the client.


## Stage III: data consolidation

Once the drop out mark has been calculated and checked by the university, we procceed to construct the consolidated spreadsheet: the spreadsheet to rule them all. So far, in each row it has a unique student record (which come from the base file), plus the drop out mark. Now we have to add more columns that can be valuable and will act as the independent variables for the model (the dependent variable, which we will try to predict, is the drop out mark). A lot of columns will not be useful, for several reasons: open questions, categorical data with too many categories, data with too many, or information that just has no relation to drop out. We explain everything in detail below.

Recall that in the last step we already did the student consolidation. Hence, when we take one student record from the base file, we know how to identify that record in other files. A useful measure to take now is to understand how much the different files match up. That is, how many unique records from the base file do not match with other files, and viceversa. The following table is an example: 

| Info file | Records present only in base file (absent from info file) | Records present only in info file (absent from base file) |
|-----------|--------------------------------------------------------|---------------------------------------------------------|
|Grades         | 1%  (334 records)    | 2%  (1.164 records)  |
|Library        | 91% (43.715 records) | 26% (1.569 records)  |
|Admission info | 41% (5.948 records)  | 4%  (402 records)    |

So for example, we have information about grades for almost all students in the base file (save for 334). And conversely, the file with the grades is useful almost in its entirety: only 2% of the records are not present in the base file (which may be duplicate records that we removed from the base file). We would like all numbers in the table to be as low as possible, albeit for different reasons

### Records present only in base file

In the middle column, we want the numbers to be low since otherwise the info file may not be so useful for the model. For example, we lack library info for 91% of students, and admission info for 41% of students. The first thing to do in these cases is to check with the university: are the info files complete? Can they get the info for the missing records? If they can, then we have to ask them to send a new and more complete version of the info file, and re-calculate the above table. It is thus an iterative process.

If they can't, then we can still take some measures. In the case of library info, for example, if a student is missing then it means he simply does not have library activity: so we can still construct some indicators (e.g. the number of books borrowed is 0). In other cases, like admission info, it could happen that they don't have the info for all students because they just recently started collecting the info, so it is only available for recent years. In such a situation, we cannot just make up the missing data. But we can still do something: in the consolidated spreadsheet, add a boolean column that indicates for each row whether the student is present in a given info file. Those indicators can be a way for the model to incorporate a file even for records that are absent from it.

### Records present only in info file

In the right column, we want numbers to be low mainly for one reason. For example, look at the library info. 26% of students in that file are absent from the main file. That is strange, since the files supposedly cover the same time period (as is requested to the university when the data origins are discussed). Could it be that those 1.569 students are missing from the base file and should be added to it? We need to look and the data and see whether we can dismiss that alternative. In this case (UdL) we can: turns out that the library info covered a longer period of time than the base file, so it naturally included some students that were absent in the main file. If such an explanation can not be found, then it needs to be comunicated to the university that the base file may be missing some student records, and continue from there.

### Exploring the data

So we have created the consolidated spreadsheet, added all unique students to it and calculated the drop out mark. We then looked at the info files and calculated to what degree they matched with the base file. Based on that, we iterated the files with the client, until we arrived at their most complete version. And we added, to the consolidated spreadsheet, columns that indicated the presence and absence of students in the different files. What now?

Well, now comes the time to include in the consolidated spreadsheet all the information, from the different files, that the model can use to predict the drop out mark. So we will add columns to the consolidated spreadsheet, containing that information. To understand how to do it, we will first give some general advice on how to explore the data, and then describe for specific files which information is usually valuable.

### General exploration of the data

Every file containing student data needs to be explored and understood. There is already a Python script that helps with this, on top of which a better tool could be built. So, a way of doing the exploration is: for each file, we iterate over all the columns, and for each column we do the following:

 1. It is important to understand what the different columns mean: the header usually helps with this but it is also important to ask the client for a brief explanation of each field. This is an iterative process, so it will be increasingly clarified as we explore the data and ask the university questions. If we don't understand what a column means, we can't verify that it looks OK. Also, it is important to be able to give the model an interpretation, mainly for two reasons. First of all, the client wants to understand, and it won't trust as much a model where the variables used are obscure. Second, the drop out risk that the model computes needs to generate concrete preventative measures, and that cannot be done if we don't understand the variables analyzed by the model.
 2. See the type of data. Is the column storing categorical data? Is it numerical data? Integer or decimal? Is it students' ids? This should be contrasted with the theoretical understanding of the column: if the header is `% of class attendance` and we see text values, or negative values, then we can suspect something is wrong, and ask the university about it. Maybe there was an error and they need to fix the file, maybe we were missing something (for example, grades systems vary from country to country), maybe something was just not clear (for example, negative values represent NaN values).
 3. If the data is numerical, make sure it makes sense. 
 4. If the data is not numerical, check whether it is categorical or not. Currently, we do not have capability to handle open questions (for more on that, see the Notes), so any column that comes from an open question field is not useful and should not be added to the consolidated spreadsheet, unless the university is willing to categorize it (this option should be discussed, but they are usually not willing to do it). If it is categorical, see how many categories there are: the R scripts can't handle more than 30 categories or so (which makes sense, more than that is probably too specific). That usually happens with information that is very low level: state where the student comes from may be of interest, but the county is too specific. Hence, if there are too many categories, either dismiss the column or cluster the categories. We mention more specific examples below, but to illustrate: if a column indicates the nationality of the student, then it is probably a good idea to cluster it into just two categories: *National* and *Foreigner*, which probably contain most of the relevant information. Also, if some categories contain very few data, then they can probably be clustered into an *Other* category.

**Notes**: there are some TODOs relative to this stage which could be very helpful and save a lot of time. I will mention them superficially now, but it is important at some point to sit down, think about them carefully, and give concrete proposals to improve the process.

- **Data exploration tools**: Fran uses SPSS, a commercial software by IBM, to explore the data (e.g. for each categorical variable, see the values and their frequencies, and maybe homologate some of them). As described above, a lot of data wrangling is done, so deciding on / developing some tools to explore the data could be useful. If SPSS is the right tool, then we can use that. Alternatively, we can construct custom tools (there is already a Python script that has some functionality, something better could be built on top). The point is: as the AI team (and not only the current members), if we really want to be comfortable with Stella, then we need some tools [plus documentation] to manage all the main tasks of a Stella process. Plus, if I may coin a tentative phrase: *tools as documentation*.
- **Outlier detection**: related to the previous suggestion, but I specify it separately since it is something that needs to be done for almost all columns. If a column has categorical data, it is useful to detect outliers (categories with very few instances) because they can point to errors in the data (categories that don't make sense) or to categories that are so unfrequent that it is better to cluster them as *Other*. If a column is numerical, then it is also useful to detect outliers, since it can also indicate errors in the data. At UdL, for instance, the *Age* column contained negative values, and also values that didn't make sense (students that were less than 10 years old), so those cases were better handled as NaN. Some simple things that the tool could do:
	- Detect categories with few instances (less than some small %).
	- Boxplot numerical data so outliers stand out immediately.
	- Detect numerical values outside of a given range (for example, at UdL grades go from 0 to 10, so anything outside that is probably an error).
	- Detect categories or values with too high a frequency. For example, at UdL there were a huge number of 0 grades, but that was because a 0 could mean: student did not attend the exam, student dropped out of the class before the exam, or student actually took the exam and got a 0. First two cases were better handled as NaN or missing data.
- **Handling open questions**: some questions are too open and difficult to handle. But others have shorter / simpler answers, which would be very easy to categorize manually, and I think they should also be easy to categorize automatically, using some pre-trained word embedding (e.g. word2vec). For example, at a UdL survey there was the question *How many siblings do you have?* and it was open, so the answers were of the form:
	- 2
	- Two
	- two
	- None
	- 0
	- only one

  Messy to look at, at first, but with very few categories in the end, and hence probably easy to handle. This connects with a different but important point: universities many times lack what we could call *data culture*. They don't really know how to use and handle data correctly. So an improtant thing that we can do is give them feedback on their data. In this case, for example, we could say to them: *When you have questions such as this one, where the answers are limited to a few categories, you should make the question categorical in the survey*. Or *Try to ask most questions in such a way that they can be answered as categories rather than open texts*. That feedback helps them, and is useful for us:
	- If we continue working with them, we will have better data to work with in the future. They won't think of this kind of improvement on their own.
	- It shows the univertity that we have experience working with this kind of project. They are dedicated to education, and need not be experts in data. But we do, since they  hired us precisely to handle all of this. So giving them feedback is a way of validating our expertise.


## Stage IV: univariate analysis

### Running the R script

### Choosing the variables of interest

Once the univariate analysis indicates which variables are more relevant, a subset of them should be chosen to generate a detailed report for the client. Some criteria to choose those variables:
- They should have prediction power. Variables with the lowest KS values are probably not interesting and should not be included in the report.
- Everyone's favorite: diversity. Usually, grades have the highest KS values, but it would not be very interesting to analyze only those variables. Hence, it is advisable to include a variety of factors, so hopefully all original files / data origins are represented in the report.
- "Production" usefulness: once the model is actually deployed, not all variables will be equally helpful. That's where the time component comes into play: the average grade of a period is really not that useful, since we only have it at the end of the period, and by then the student may be on the verge of desertion, or have already dropped out. So it makes sense to include variables that are available early in a period (if they have a good KS value), because they can actually provide early alerts: for example, **the first grade of the period, rather than the last**. 
- Regarding the previous example, it is usually the last grade of the period that will have a higher KS value, but bear in mind that that can be deceiving. Some variables may seem to have a lot of explanatory power, but only because they are more consequence of desertion than cause. Low % of class assistance, minimum grades or low curricular progress may have only ex-post explanatory power, and be more dependent than independent: of course a student that deserted during a period is going to have low class assistance during that period. This doesn't mean that those kind of variables should be dismissed at once, but it is important to check that they can actually be used to explain desertion.
- Their relevance should be easy to explain to the client. For example, it is usually better for the model to use a relative ranking for grades than the absolute number (for example, *student is in the bottom 15%* rather than *student got a 2.5 out of 10*), but a ranking is harder for the client to handle.

### Generating the report

### Discussing with the university

Once the report is completed, it must be checked to make sure it makes sense. It seems  strange that students that do well in the first grade of the period turn out to drop out in higher proportion than the others. It is not impossible, but before discussing the report with the client, we must analyze it ourselves, and everything that seems weird must be checked and, in case it looks error-free to us, make a point of discussing it with the university: could there be errors in the data? Is there some phenomenon that we are not aware of? Maybe students that are doing well drop out more because they get good offers from other universities, who knows.

Also, before discussing the report with the university, we need to draw some general interpretation from it, and understand what the data is saying. So we need to go from chart to chart, try to cluster the results and data in a few general conclusions. When doing this, consider the following:
- Different charts may be speaking about the same phenomenon and need to be clustered in the interpretation. For example, 
- Refer to past experiences, since projects tend to have many similarities (e.g. grades are always relevant).

Once we feel that we understand the report well, and are comfortable with the conclusions that arise from it, a meeting has to be held with the university, which constitutes the **third checkpoint**. The main purpose of the meeting is to explain the work done so far. So we show them the report, explain how to read the charts, and tell them the conclusions that we derive from the univariate analysis. For example, in the case of UdL the conclusions were something like: 

> The main predictor of desertion is academic performance: level of curricular progress, number of failed classes so far and, especially, grades. Even early in the period, with the first batch of grades, we can have a rough idea of which students are at risk. We also note that students that enroll at an older age (over 19) have a lower desertion risk. Finally, there are a number of factors that show a smaller but still significant effect: library activity, sex, and the psychological evaluation done at admission all are relevant to predicting drop out likelihood.

All of this conclusions, of course, must have their counterpart and be supported by the different charts of the report, and our exploration of the data, so they will have validity in the eyes of the client. Also, there may be variables that they are interested in but we dismissed: make that clear to them, and offer to complement the report with any variables that they want.

After explaining the report in detail, and clearing up the client's questions, there are two possible courses of action, which must be discussed with the university. It is they that ultimately decide. The first option is to go deeper into the univariate analysis: look at the data again and in more detail, get more refined charts and categories, maybe categorize some of the open questions. Some universities ascribe a lot of value to this sort of analysis, since it is information that they don't usually have. The second option is to go straight into the multivariate analysis: some clients will be anxious for results (a final model) and will prefer this alternative.


**Notes**: there are some TODOs relative to this stage which could be very helpful and save a lot of time. I will mention them superficially now, but it is important at some point to sit down, think about them carefully, and give concrete proposals to improve the process.

- **Process standards**: I'm not sure about this, but I think maybe the Stella process could be standardized a bit more. For example, as we describe above, at the third checkpoint (the meeting to explain univariate analysis) there are two courses of action, and it is mainly the university that decides which one to choose. The first one requires more man-hours and implies a longer overall process, while the second doesn't. That uncertainty makes it harder to predict the effort required by a project, which could be problematic for the planning process in Foris.
- **Report generation tools**: we already described what kind of reporting we need to generate at this stage. Most of it is pretty standard, so it is possible to automatize a large part of the work and save a lot of time for future projects. This TODO will probably be repeated for all stages of the process where some reporting needs to be done. Some basic things that the tool could do (there is already a Python script that covers the first two points decently, and the third and fourth points partially, but it should be improved and made more portable as a tool):
	- Categorize numerical data (very important, otherwise is difficult to show the impact of numerical data in a readable and interpretable way).
	- Generate the charts and tables that are traditionally used for univariate analysis.
	- Include a friendly way of previewing charts / tables and do some fixing / prettifying to them (e.g. fix the label placement), since a standard tool will probably create display problems for some charts.
	- Allow for some light customization of the charts / tables. For example, for some fields (categorical data) we may want to print the categories in order of most frequent to less frequent, for other fields (categorized numerical data) we will want to print / plot the categories in numerical ascending order, and for both cases there may be exceptions.
	- Generate a general report that groups all charts / data into one document (a PDF, or a presentation).
