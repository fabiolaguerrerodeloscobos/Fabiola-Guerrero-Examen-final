# Fabiola-Guerrero-Examen-final
---
title: "Examen Fabiola Guerrero"
author: "Fabiola Guerrero"
date: "2024-05-16"
output:
  html_document: default
  pdf_document: default
---
```{r}
library (pacman)
p_load("vroom",
       "dplyr",
       "ggplot2",
       "ggrepel",
       "matrixTests") #Para analisis estad?stico
```

```{r}
Volcano_data <- vroom(file="https://raw.githubusercontent.com/ManuelLaraMVZ/Transcript-mica/main/Datos%20completos%20miRNAs%20ejercicio.csv")
head(Volcano_data)

Volcano_data2 <- Volcano_data %>% 
  filter(Type == "Target") %>% 
  select(1,3,4,5,6,7,8)
```

```{r}
Control <- Volcano_data %>% 
  filter(Type == "Selected Control") %>% 
  select(1,3,4,5,6,7,8)
```

```{r}
meanT1 <- mean(Control$T1)
meanT2 <- mean(Control$T2)
meanT3 <- mean(Control$T3)
meanC1 <- mean(Control$C1)
meanC2 <- mean(Control$C2)
meanC3 <- mean(Control$C3)

DCT <- Volcano_data2 %>% 
  mutate(DCT1 = T1-meanT1, DCT2 = T2-meanT2, DCT3 = T3-meanT3,
         DCC1 = C1-meanC1, DCC2 = C2-meanC2, DCC3 = C3-meanC3)
```

```{r}
DosDCT <- DCT %>% 
  mutate(A = 2^-(DCT1), B= 2^-(DCT2), C= 2^-(DCT3), 
         D = 2^-(DCC1), E= 2^-(DCC2), F= 2^-(DCC3)) %>% 
  select(1,A,B,C,D,E,F)
```

```{r}
tvalue <- row_t_welch(DosDCT[,c("A","B","C")], DosDCT[,c("D","E","F")])

View(tvalue)

FCyPV <- DosDCT %>% 
  mutate(pvalue = tvalue$pvalue) %>% 
  mutate(FCTx=((A+B+C)/3)) %>% 
  mutate(FCCx=((D+E+F)/3)) %>% 
  mutate(FC =(FCTx/FCCx)) %>% 
  select(1,pvalue,FC)
```

```{r}
p_val = 0.05

fchange_threshold=2
```

```{r}
ValoresVolcanoP <- FCyPV %>% 
  mutate(LPV = -log10(pvalue),
         LFC = log2(FC)) 
ValoresVolcanoP
```


```{r}
downregulated = ValoresVolcanoP %>% 
  filter (LFC < -log2(fchange_threshold)) %>% 
  filter (pvalue < p_val)
```

```{r}
upregulated = ValoresVolcanoP %>% 
  filter (LFC > log2(fchange_threshold)) %>% 
  filter (pvalue < p_val)
```

```{r}
top.down <- downregulated %>% 
  arrange(pvalue) %>% 
  head(5)
```

```{r}
top.up <- upregulated %>% 
  arrange(pvalue) %>% 
  head(5)
```


```{r}
Volcano <- ggplot(data = ValoresVolcanoP,
                   mapping = aes(x = LFC,
                                 y = LPV))+
  geom_point(alpha = 0.8,
             color = "black")+
  labs(title = "Fabiola Guerrero")+
  theme_classic()+
  xlab("Log2(FC)")+
  ylab("-Log10(p-value)")+
  geom_hline(yintercept = -log10(p_val),
             linetype = "dashed")+
  geom_vline(xintercept = c(-log2(fchange_threshold),log2(fchange_threshold)),
             linetype = "dashed")+
  geom_point(data=upregulated,
             x=upregulated$LFC,
             y = upregulated$LPV,
             alpha=1,
             size = 3,
             color = "#E64B35B2")+
  geom_point(data=downregulated,
             x=downregulated$LFC,
             y=downregulated$LPV,
             alpha = 1,
             size = 3,
             color = "#3C5488B2")+
  geom_label_repel(data=top.up,
                   mapping = aes(x= top.up$LFC,
                                 y= top.up$LPV),
                   label = top.up$gene,
                   max.overlaps = 50)+
  geom_label_repel(data=top.down,
                   mapping = aes(x = top.down$LFC,
                                 y = top.down$LPV),
                   label =top.down$miRNA,
                   max.overlaps = 50)
Volcano

```

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

## R Markdown

This is an R Markdown document. Markdown is a simple formatting syntax for authoring HTML, PDF, and MS Word documents. For more details on using R Markdown see <http://rmarkdown.rstudio.com>.

When you click the **Knit** button a document will be generated that includes both content as well as the output of any embedded R code chunks within the document. You can embed an R code chunk like this:

```{r cars}
summary(cars)
```

## Including Plots

You can also embed plots, for example:

```{r pressure, echo=FALSE}
plot(pressure)
```

Note that the `echo = FALSE` parameter was added to the code chunk to prevent printing of the R code that generated the plot.
