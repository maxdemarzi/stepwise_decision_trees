# stepwise_decision_trees
POC of stepping through a decision tree one

This project requires Neo4j 3.4.x or higher

Instructions
------------ 

This project uses maven, to build a jar-file with the procedure in this
project, simply package the project with maven:

    mvn clean package

This will produce a jar-file, `target/decision_trees_with_rules-1.0-SNAPSHOT.jar`,
that can be copied to the `plugin` directory of your Neo4j instance.

    cp target/decision_trees_with_rules-1.0-SNAPSHOT.jar neo4j-enterprise-3.4.8/plugins/.
    

Download and Copy two additional files to your Neo4j plugins directory:

    http://central.maven.org/maven2/org/codehaus/janino/commons-compiler/3.0.8/commons-compiler-3.0.8.jar
    http://central.maven.org/maven2/org/codehaus/janino/janino/3.0.8/janino-3.0.8.jar
   

Restart your Neo4j Server.

Create the Schema by running this stored procedure:

    CALL com.maxdemarzi.schema.generate
    

Create some test data:

    CREATE (tree:Tree { id: 'bar entrance' })
    CREATE (over21_rule:Rule { name: 'Over 21?', parameter_names: 'age', parameter_types:'int', expression:'age >= 21' })
    CREATE (gender_rule:Rule { name: 'Over 18 and female', parameter_names: 'age,gender', parameter_types:'int,String', expression:'(age >= 18) && gender.equals(\"female\")' })
    CREATE (answer_yes:Answer { id: 'yes'})
    CREATE (answer_no:Answer { id: 'no'})
    CREATE (tree)-[:HAS]->(over21_rule)
    CREATE (over21_rule)-[:IS_TRUE]->(answer_yes)
    CREATE (over21_rule)-[:IS_FALSE]->(gender_rule)
    CREATE (gender_rule)-[:IS_TRUE]->(answer_yes)
    CREATE (gender_rule)-[:IS_FALSE]->(answer_no)    
    CREATE (age:Parameter {name:'age', type:'int', prompt:'How old are you?', expression:'(age > 0) &&  (age < 150)'})
    CREATE (gender:Parameter {name:'gender', type:'String', prompt:'What is your gender?', expression: '\"male\".equals(gender) || \"female\".equals(gender)'} )
    CREATE (over21_rule)-[:REQUIRES]->(age)
    CREATE (gender_rule)-[:REQUIRES]->(age)
    CREATE (gender_rule)-[:REQUIRES]->(gender)

Try it:

    CALL com.maxdemarzi.stepwise.decision_tree('bar entrance', {});
    CALL com.maxdemarzi.stepwise.decision_tree('bar entrance', {age:'20'});
    CALL com.maxdemarzi.stepwise.decision_tree('bar entrance', {age:'24'}); 
    CALL com.maxdemarzi.stepwise.decision_tree('bar entrance', {gender:'male', age:'20'});
    CALL com.maxdemarzi.stepwise.decision_tree('bar entrance', {gender:'female', age:'19'});
    CALL com.maxdemarzi.stepwise.decision_tree('bar entrance', {gender:'male', age:'23'});     

Create some more test data:


    CREATE (tree:Tree { id: 'funeral' })
    CREATE (good_man_rule:Rule { name: 'Was Lil Jon a good man?', parameter_names: 'answer_1', parameter_types:'String', script:'switch (answer_1) { case \"yeah\": return \"OPTION_1\"; case \"what\": return \"OPTION_2\"; case \"okay\": return \"OPTION_3\"; default: return \"UNKNOWN\"; }' })
    CREATE (good_man_two_rule:Rule { name: 'I said, was he a good man?', parameter_names: 'answer_2', parameter_types:'String', script:'switch (answer_2) { case \"yeah\": return \"OPTION_1\"; case \"what\": return \"OPTION_2\"; case \"okay\": return \"OPTION_3\"; default: return \"UNKNOWN\"; }' })
    CREATE (rest_in_peace_rule:Rule { name: 'May he rest in peace', parameter_names: 'answer_3', parameter_types:'String', script:'switch (answer_3) { case \"yeah\": return \"OPTION_1\"; case \"what\": return \"OPTION_2\"; case \"okay\": return \"OPTION_3\"; default: return \"UNKNOWN\"; } ' })
    CREATE (answer_correct:Answer { id: 'correct'})
    CREATE (answer_incorrect:Answer { id: 'incorrect'})
    CREATE (answer_unknown:Answer { id: 'unknown'})
    CREATE (tree)-[:HAS]->(good_man_rule)
    CREATE (good_man_rule)-[:OPTION_1]->(answer_incorrect)
    CREATE (good_man_rule)-[:OPTION_2]->(good_man_two_rule)
    CREATE (good_man_rule)-[:OPTION_3]->(answer_incorrect)
    CREATE (good_man_rule)-[:UNKNOWN]->(answer_unknown)    
    CREATE (good_man_two_rule)-[:OPTION_1]->(rest_in_peace_rule)
    CREATE (good_man_two_rule)-[:OPTION_2]->(answer_incorrect)
    CREATE (good_man_two_rule)-[:OPTION_3]->(answer_incorrect)
    CREATE (good_man_two_rule)-[:UNKNOWN]->(answer_unknown)    
    CREATE (rest_in_peace_rule)-[:OPTION_1]->(answer_incorrect)
    CREATE (rest_in_peace_rule)-[:OPTION_2]->(answer_incorrect)
    CREATE (rest_in_peace_rule)-[:OPTION_3]->(answer_correct)
    CREATE (rest_in_peace_rule)-[:UNKNOWN]->(answer_unknown)
    CREATE (parameter1:Parameter {name:'answer_1', type:'String', prompt:'What is the first answer?', expression:'\"yeah\".equals(answer_1) || \"what\".equals(answer_1) || \"okay\".equals(answer_1) || \"\".equals(answer_1)'})
    CREATE (parameter2:Parameter {name:'answer_2', type:'String', prompt:'What is the second answer?', expression:'\"yeah\".equals(answer_2) || \"what\".equals(answer_2) || \"okay\".equals(answer_2) || \"\".equals(answer_2)'})
    CREATE (parameter3:Parameter {name:'answer_3', type:'String', prompt:'What is the third answer?', expression:'\"yeah\".equals(answer_3) || \"what\".equals(answer_3) || \"okay\".equals(answer_3) || \"\".equals(answer_3)'})
    CREATE (good_man_rule)-[:REQUIRES]->(parameter1)
    CREATE (good_man_two_rule)-[:REQUIRES]->(parameter2)
    CREATE (rest_in_peace_rule)-[:REQUIRES]->(parameter3)

Try it:

    CALL com.maxdemarzi.stepwise.decision_tree('funeral', {});
    CALL com.maxdemarzi.stepwise.decision_tree('funeral', {answer_1:'what'});
    CALL com.maxdemarzi.stepwise.decision_tree('funeral', {answer_1:'yeah', answer_2:'yeah', answer_3:'yeah'});    
    CALL com.maxdemarzi.stepwise.decision_tree('funeral', {answer_1:'what', answer_2:'', answer_3:''});    
    CALL com.maxdemarzi.stepwise.decision_tree('funeral', {answer_1:'what', answer_2:'yeah', answer_3:'okay'});
    
Validate Parameters:

    CALL com.maxdemarzi.stepwise.validate('age', '20');
    CALL com.maxdemarzi.stepwise.validate('age', '-45');
    CALL com.maxdemarzi.stepwise.validate('age', 'cat');
    CALL com.maxdemarzi.stepwise.validate('gender', 'attack helicopter');    
