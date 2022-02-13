## Video Game Sales Analysis using MySQL

You can use the [editor on GitHub](https://github.com/jchen64581/jchen.github.io/edit/main/README.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

### About Dataset

The dataset is sourced from kaggle (https://www.kaggle.com/gregorut/videogamesales ) which contains a list of video games with sales from 1980 to 2020. The dataset has 11 columns and 16598  records. I decided to drop records with null values in order to make the result of the analysis more accurate. In addition, I split the dataset into two related tables based on the principle of data normalization - the "game_info" table contains the basic information of the game, and the "game_sales" table contains the sales records of the game in various regions. 

### Using Temporary Table
Since we will need to join both tables multiple times to perform analysis, I would create a temporary table for resusiblity purpose. Using temporary can make the query a lot shorter and also speed up the runtime to increase efficiency.
``` sql
CREATE TEMPORARY TABLE game_sales_temp
SELECT
	i.game_id,
    name,
    platform,
    year,
    genre,
    publisher,
    na_sales,
    eu_sales,
    jp_sales,
    other_sales,
    global_sales
FROM game_info i
JOIN game_sales s
	ON i.game_id = s.game_id;

SELECT * FROM game_sales_temp;

[Link](url) and ![Image](src)
``` sql

For more details see [Basic writing and formatting syntax](https://docs.github.com/en/github/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/jchen64581/jchen.github.io/settings/pages). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://support.github.com/contact) and we’ll help you sort it out.
