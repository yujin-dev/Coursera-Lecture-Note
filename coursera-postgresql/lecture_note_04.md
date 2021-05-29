## regular expressions
`select email from em where email ~ 'umich';` : like : contains umich  
`select email from em where email ~ '^c';` like : starts with c  
`select email from em where email ~ 'edu$';` like : ends with edu  
`select email from em where email ~ '^[gnt]';` like : start with g OR n OR t
`select email from em where email ~ '[0-9]';` like : contains 0 ~ 9 ( any )    
`select email from em where email ~ '[0-9][0-9]';` like : contains 0~9 and 0~9

