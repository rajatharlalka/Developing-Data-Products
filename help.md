### Usage:
1. Just choose the **outcome** from the Outcome panel on the right.
2. Select any number of **predictors** from the Predictor panel on the right.
 * __*Note:*__ Off-course including the outcome in the predictors choice have no sense.

### Output:
* You will see on the right panel three items:
 1. Pairwise plot for the **outcome** and all **predictors** you choose.
 2. A summary of a linear model fit with the outcome and predictors you choose.
 3. A subset of the **mtcars** data frame where the __outcome__ in the first column followed by the __predictors__ in the rest of the columns.
 
### Ui.R Code

```r
#ui.R
require(shiny)
require(markdown)
require(knitr)

data(mtcars)

shinyUI(
  pageWithSidebar(
    headerPanel("Fitting mtcars variables and predictors"),
    sidebarPanel(
      h2("Introduction"),
      p("This application fits any outcome in the mtcars data frame with any number of 
        predictors within the mtcars. It visualize the output and predictors relationship in scatter matrix 
        and the linear model summary is displayed below the scatter matrix."),
      h2("Choosing Outcome and Predictors"),
      h3("Chossing Outcome"),
      selectInput("outcome", "Choose Outcome", colnames(mtcars), selected = "mpg", multiple = FALSE,
                  selectize = TRUE),
      h3("Chossing predictors"),
      checkboxGroupInput("predictors","Choose Predictors",colnames(mtcars),selected = "wt"),
      includeMarkdown(path="./help.Rmd")
      
    ),    
    mainPanel(
      h2("Scattar Plot and Model Results"),
      h3("Scatter Plot"),
      plotOutput('pair_plot'),
      h3("Model Results"),
      verbatimTextOutput("my_model"),
      h3("Data"),
      verbatimTextOutput("df")
      
    )
  )
  
)

```



### Server.R Code


```r


prepare_data_frame <- function(outcome, predictors) {
    df <- cbind(mtcars[, outcome], mtcars[, predictors])
    df <- as.data.frame(df)
    colnames(df) <- c(outcome, predictors)
    return(df)
}

require(shiny)
shinyServer(function(input, output) {
    output$y <- renderPrint({
        input$outcome
    })
    output$x <- renderPrint({
        input$predictors
    })
    output$df <- renderPrint({
        prepare_data_frame(input$outcome, input$predictors)
    })
    output$pair_plot <- renderPlot({
        df1 <- prepare_data_frame(input$outcome, input$predictors)
        pairs(df1)
    })
    output$my_model <- renderPrint({
        f0 <- paste(input$outcome, "~", paste(input$predictors, collapse = "+"), 
            sep = "")
        f1 <- formula(f0)
        lm1 <- lm(f1, data = mtcars)
        summary(lm1)
    })
    
})

```

