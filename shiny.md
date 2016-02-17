---
title       : New Shiny Features
subtitle    : 
author      : Alex Shum
job         : 
framework   : revealjs        # {io2012, html5slides, shower, dzslides, ...}
highlighter : highlight.js  # {highlight.js, prettify, highlight}
hitheme     : tomorrow      # 
widgets     : [mathjax]            # {mathjax, quiz, bootstrap}
mode        : selfcontained # {standalone, draft}
knit        : slidify::knit2slides
---

## Shiny Developer's Conference

Alex Shum

February 17, 2016

---

## Shiny: New Features
* Dashboards
* Functional programming and reactives
* Shiny Modules
* HTML Templates
* Other

--- &vertical

## Dashboard features
* Notifications

![](https://rstudio.github.io/shinydashboard/images/menu-notifications.png)

* Task/progress bars

![](https://rstudio.github.io/shinydashboard/images/menu-tasks.png)

***

* infoboxes

![](https://rstudio.github.io/shinydashboard/images/body-infoboxes.png)

* valueboxes

![](https://rstudio.github.io/shinydashboard/images/body-valueboxes.png)

***


```r
ui <- dashboardPage(
  dashboardHeader(
    title = "Basic dashboard",
    dropdownMenu(type = "messages",
      messageItem(
        from = "Sales Dept",
        message = "Sales are steady this month."
      ),
      messageItem(
        from = "New User",
        message = "How do I register?",
        icon = icon("question"),
        time = "13:45"
      ),
      messageItem(
        from = "Support",
        message = "The new server is ready.",
        icon = icon("life-ring"),
        time = "2014-12-01"
      )
    )
  ),
  dashboardSidebar(),
  dashboardBody(
    # Boxes need to be put in a row (or column)
    fluidRow(
      box(plotOutput("plot1", height = 250)),

      box(
        title = "Controls",
        sliderInput("slider", "Number of observations:", 1, 100, 50)
      )
    )
  )
)
```

***


```r
server <- function(input, output) {
  histdata <- rnorm(500)

  output$plot1 <- renderPlot({
    data <- histdata[seq_len(input$slider)]
    hist(data)
  })
}
```

--- &vertical

## Reactive programming
* Reactive expressions are basically functions with caching.
* Use a reactive expression when you would use a function in base R.
* Reactive expressions update when inputs change.
* Reactive expressions cache results!

*** 

## Functional programming and side effects.
- A function that modifies the state outside the function.
- Pure functions return a value and make no changes to the outside state.
- Reasoning through code is easier with pure functions.  
- Reactive expressions are basically functions: avoid side effects

***


```r
#incorrect 
data <- reactive({
  x = slow(head(x_data, input$obs))
  y = head(y_data, input$obs)
  df = data.frame(x = x, y = y)

  output$table <- renderTable({
    df
  })

  df
})
```

***


```r
#correct
data <- reactive({
  x = slow(head(x_data, input$obs))
  y = head(y_data, input$obs)
  data.frame(x = x, y = y)
})  

output$table <- renderTable({
  data()
})
```

--- &vertical

## Shiny Modules
- Modularize shiny code
- Wrap UI elements in a function
- New functionality to insure namespace uniqueness of input/output IDs

***


```r
# Module UI function
csvFileInput <- function(id, label = "CSV file") {
  # Create a namespace function using the provided id
  ns <- NS(id)

  tagList(
    fileInput(ns("file"), label),
    checkboxInput(ns("heading"), "Has heading"),
    selectInput(ns("quote"), "Quote", c(
      "None" = "",
      "Double quote" = "\"",
      "Single quote" = "'"
    ))
  )
}
```

***


```r
# Module server function
csvFile <- function(input, output, session, stringsAsFactors) {
  userFile <- reactive({
    validate(need(input$file, message = FALSE))
    input$file
  })

  dataframe <- reactive({
    read.csv(userFile()$datapath,
      header = input$heading,
      quote = input$quote,
      stringsAsFactors = stringsAsFactors)
  })

  observe({
    msg <- sprintf("File %s was uploaded", userFile()$name)
    cat(msg, "\n")
  })

  # Return the reactive that yields the data frame
  return(dataframe)
}
```

***


```r
server <- function(input, output, session) {
  datafile <- callModule(csvFile, "datafile",
    stringsAsFactors = FALSE)

  output$table <- renderDataTable({
    datafile()
  })
}

shinyApp(ui, server)
```

***


```r
ui <- fluidPage(
  sidebarLayout(
    sidebarPanel(
      csvFileInput("datafile", "User data (.csv format)")
    ),
    mainPanel(
      dataTableOutput("table")
    )
  )
)
```

--- &vertical

## Shiny Default UI
* User interface built in R uses bootstrap.
* Sensible defaults but limited functionality.
* `fluidPage` creates a bootstrap 12-column layout
  - `fluidRow` and `column` correspond to `<div-class="row">` and `<div class="col-">` in bootstrap
* `sidebarLayout` premade template with a 4-column sidebarPanel and 8-column mainPanel.


```r
shinyUI(fluidPage(
  titlePanel("title panel"),

  sidebarLayout(
    sidebarPanel( "sidebar panel"),
    mainPanel("main panel")
  )
))
```

***

## HTML Templates

```
<!DOCTYPE html>
<!-- template.html -->
<html>
  <head>
    {\{ headContent() }\}
  </head>
  <body>
    <div>
      {\{ button }\}
      {\{ slider }\}
   </div>
  </body>
</html>
```

***

## HTML Template from Shiny

```r
htmlTemplate("template.html",
  button = actionButton("action", "Action"),
  slider = sliderInput("x", "X", 1, 100, 50)
)
```

---

## Other new features
- Easy deployment through shinyapps.io
- Shiny gadgets: apps packaged as functions that will return a value to the caller.
- Grid Style sheet layout for shiny apps.
- DataTable javascript library native compatability in shiny.
- New debugging and profiling tools.

---

## END
