#  Intro to Data Science in Econ 
#AVIATION DELAY ANALYSIS - November 2025
library(broom)
library(tidyverse)
library(corrplot)
library(readxl)
library(plotly)
library(scales)

df <- read_csv("Aviation_Delay_causes_Final_Project_VF.csv")
Download_Column_Definitions <- read_excel("Download_Column_Definitions.xlsx")

#. . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . .####
#NIP/DATA CLEANING
df_data_cleaned <- df %>%
  filter(arr_flights > 0) %>%
  mutate(
    avg_arr_delay   = round(arr_delay / arr_flights, 4),      #total delay time (mins) / total flights
    delay_rate      = round(arr_del15 / arr_flights, 4),      #total delay flights (over 15 mins)/ total flights
    total_delay_min = carrier_delay + weather_delay + nas_delay + security_delay + late_aircraft_delay
  ) %>%
  mutate(
    # Each pct_X = share of that cause out of total delay minutes
    pct_carrier       = round(carrier_delay / total_delay_min, 4),  
    pct_weather       = round(weather_delay / total_delay_min, 4),
    pct_nas           = round(nas_delay / total_delay_min, 4),
    pct_security      = round(security_delay / total_delay_min, 4),
    pct_late_aircraft = round(late_aircraft_delay / total_delay_min, 4)
  )





#. . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . .####

#(CLICK1)####
# Research Q1: What is the biggest driver of arrival delay?####
#Biggest Driver of Delay

# Total minutes and percentage per cause

cause_totals <- df_data_cleaned %>%
  summarise(
    Carrier = sum(carrier_delay),
    Weather = sum(weather_delay),
    NAS = sum(nas_delay),
    Security = sum(security_delay),
    Late_Aircraft = sum(late_aircraft_delay)
  ) %>%
  pivot_longer(everything(), names_to = "cause", values_to = "total_minutes") %>% # Change from 2 row/5 columns ->  5 rows/2 columns for ggplot
  mutate(
    pct   = total_minutes / sum(total_minutes) * 100,
    cause = fct_reorder(cause, total_minutes)
  )

print(cause_totals)

##PLOT 1####

ggplot(
  cause_totals,
  aes(
    y = total_minutes / 1000000, # Convert mins to millions for cleaner y-axis
    x = cause
  )
) +
  geom_col(width = 0.65) +
  geom_text(
    aes(label = paste0(round(pct, 1), "%")),
    vjust = -0.5,
    size = 4
  ) +
  labs(
    title = "Total Flight Delay Minutes by Cause",
    subtitle = "November 2025 Breakdown",
    x = "Cause",
    y = "Delay Minutes (Millions)"
  ) +
  theme_minimal(base_size = 13) +
  theme(
    plot.title = element_text(face = "bold", size = 16),
    plot.subtitle = element_text(color = "gray40"),
    axis.text.x = element_text(face = "bold"),
    panel.grid.major.x = element_blank() # remove vertical gridline
  )

#. . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . .####



#(CLICK2)####
#Research Q2: Do busier airports have worse delays?####
#Size of Airports vs Delays plots

airport_df <- df_data_cleaned %>%
  group_by(airport) %>%
  summarise(
    arr_flights = sum(arr_flights, na.rm = TRUE),
    arr_delay = sum(arr_delay, na.rm = TRUE)
  ) %>%
  mutate(
    avg_delay  = arr_delay / arr_flights
  )



##PLOT 2####
ggplot(airport_df, aes(x = arr_flights, y = avg_delay)) +
  geom_point(aes(size = arr_flights), #Here make the bubble size depends on the size of the airports
             color = "steelblue", 
             alpha = 0.25) +  
  geom_smooth(method = "lm", 
              color = "firebrick",
              se = FALSE) +
  scale_x_continuous(labels = label_comma()) + 
  labs(
    title = "Does Airport Volume Predict Delays?",
    subtitle = "Relationship between Total Arrivals and Average Delay per Flight",
    caption = "Data: November 2025",
    x = "Total Arrivals",
    y = "Avg Delay (min/flight)",
    size = "Flight Volume"
  ) +
  theme_minimal(base_size = 12) +
  theme(
    plot.title = element_text(face = "bold"),
    legend.position = "bottom"
  )
#Each dot (or bubble) on this graph represents a single, unique airport in the dataset for the month of November 2025.


##**Summary of Linear Model####
ghj <- summary(lm(avg_delay ~ arr_flights, data = airport_df))
ghj




#. . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . .####


#(CLICK3)####
#Average Arrival Delay by Carrier/Interactive Plot####

#Research Q3: Which carriers have the worst delays, and what is causing them####
#Carrier Comparison

carrier_df <- df_data_cleaned %>%
  group_by(carrier, carrier_name) %>%
  summarise(
    avg_delay = sum(arr_delay) / sum(arr_flights),
    pct_carrier = sum(carrier_delay) / sum(total_delay_min),
    pct_weather = sum(weather_delay) / sum(total_delay_min),
    pct_nas = sum(nas_delay) / sum(total_delay_min),
    pct_late_aircraft = sum(late_aircraft_delay) / sum(total_delay_min),
    pct_security = sum(security_delay) / sum(total_delay_min),
  )


## PLOT_int 3 Avg Delay by Carrier####
p1 <- ggplot(carrier_df, 
             aes(x = reorder(carrier, -avg_delay),  # sort the X (carrier) order by their delay high to low
                 y = avg_delay,
                 text = paste0(carrier_name, 
                               "\nAvg Delay: ", round(avg_delay, 1), " mins"))) + #hover tool
  geom_col(fill = "tomato") +
  labs(
    title = "Q3 - Avg Arrival Delay by Carrier (Nov 2025)",
    x     = "Carrier",
    y     = "Avg Delay/per flight (min)"
  ) +
  theme_minimal() +
  scale_y_continuous(breaks = seq(0, 40, by = 5)) +
  theme(
    panel.grid.minor.y = element_line(color = "gray92")
  )
ggplotly(p1, tooltip = "text") # Display the paste0() tooltip on hover instead of raw variable name



#. . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . .####
#(CLICK4)####
#Delay Cause Mix by Carrier/Interactive Plot####
#Research Q4: so what contribute to G7's delay the most?####

carrier_long <- carrier_df %>%
  select(carrier, carrier_name, avg_delay, 
         pct_carrier, pct_weather, pct_nas, pct_late_aircraft, pct_security) %>%
  pivot_longer(
    cols      = c(pct_carrier, pct_weather, pct_nas, pct_late_aircraft, pct_security),
    names_to  = "cause",
    values_to = "pct"
  ) %>%
  mutate(cause = case_when(
    cause == "pct_carrier" ~ "Carrier",
    cause == "pct_weather" ~ "Weather",
    cause == "pct_nas" ~ "NAS",
    cause == "pct_late_aircraft" ~ "Late Aircraft",
    cause == "pct_security" ~ "Security"
  ))
##PLOT 4####
p2 <- ggplot(carrier_long,
             aes(x = reorder(carrier, -avg_delay),
                 y = pct,
                 fill = cause,
                 text = paste0(carrier_name,
                               "\nCause: ",  cause,
                               "\nShare: ",  percent(pct, accuracy = 0.1)))) +
   geom_col(position = "stack") +
  scale_y_continuous(labels = percent_format()) +
  labs(
    title = "Q4 - Delay Cause Mix by Carrier (Nov 2025)",
    x     = "Carrier (sorted worst to best)",
    y     = "Share of Delay Minutes",
    fill  = "Cause"
  ) +
  theme_minimal()

ggplotly(p2, tooltip = "text")






#carrier_lookup
carrier_lookup <- df_data_cleaned %>%
  distinct(carrier, carrier_name)
carrier_lookup










#. . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . .####





#(CLICK5)####
#Research Q5: Weather vs. NAS delay — are they correlated?####

airport_q5 <- df_data_cleaned %>%
  group_by(airport) %>%
  summarise(
    # total_flights = sum(arr_flights),
    avg_weather   = sum(weather_delay) / sum(arr_flights),
    avg_nas       = sum(nas_delay)     / sum(arr_flights)
  )


# (CLICK)Scatter plot_Weather vs NAS Delay Per flight
ggplot(airport_q5, aes(x = avg_weather, y = avg_nas)) +
  geom_point(color = "purple", 
             alpha = 0.6,) +
  geom_smooth(method = "lm", 
              color = "darkorange",
              se = FALSE) +
  labs(
    title = "Q5 - Weather vs. NAS Delay per Flight",
    x     = "Avg Weather Delay (min)",
    y     = "Avg NAS Delay (min)"
  ) +
  theme_minimal()


##Correlation Test####

cor.test(airport_q5$avg_weather, airport_q5$avg_nas)
#p-Value is well below 0.05
#r=0.192, weak to moderate positive






##Correlation matrix across all delay causes####
delay_cols <- df_data_cleaned %>%
  select(carrier_delay, 
         weather_delay, 
         nas_delay,
         security_delay, 
         late_aircraft_delay)

cor_matrix <- cor(delay_cols, use = "complete.obs")   
round(cor_matrix, 3)

##(CLICK)PLOT 5####
corrplot(cor_matrix,
         method      = "color",
         type        = "upper",
         addCoef.col = "white",      
         number.cex  = 1.2,          
         tl.col      = "black",
         tl.srt      = 45)

mtext("Q5 - Delay Cause Correlation Matrix", 
      side = 3,      # top of plot
      line = 3,      # how far above 
      cex  = 1.3, 
      font = 2)



