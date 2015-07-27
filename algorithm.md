## Running algorihtms
To run an algorithm in jMetal you have two choices: using an IDE or using the command line. We explain both methods in this section. We comment first how configure a metaheuristic to solve a problem.

### Configuring algorithms
In jMetal 5 the only to configure an algorithm is to write a class for that purpose; we refer to those classes as runners. Doing that from an external configuration file is a issue belonging to the todo list of the next release.

We provide at least a runner algorithm. They can be found in the `jmetal-exec` module, in folder https://github.com/jMetal/jMetal/tree/master/jmetal-exec/src/main/java/org/uma/jmetal/runner/multiobjective. 

For example, in the case of NSGA-II we include up to 5 runners:
* `NSGAIIRunner`: configuration of the standard NSGA-II to solve continuous problems.
* `NSGAIIInteger`: configuration to solve integer problems.
* `NSGAIIBinary`: configuration to solve binary problems.
* `NSGAIIMeasures`: similar to `NSGAIIRunner`, but it includes examples of how to use measures.
* `ParallelNSGAII`: as `NSGAIIRunner` but configured to use threads to evaluate the populations in parallel.

We describe next the `NSGAIIRunner` class. The Javadoc comment indicate the program parameters: the first one is the class of the problem to solve and, optionally, a reference front file name can be given. In that case, the reference front will be used to compute all the quality indicators available:
```java
public class NSGAIIRunner extends AbstractAlgorithmRunner {
  /**
   * @param args Command line arguments.
   * @throws JMetalException
   * @throws FileNotFoundException
   * Invoking command: java org.uma.jmetal.runner.multiobjetive.NSGAIIRunner problemName [referenceFront]
   */
  public static void main(String[] args) throws JMetalException, FileNotFoundException {
```
The first part of the `main` method is for declaring the types of the problem to solve (a problem dealing with `DoubleSolution` individuals) and the operators. The `referenceParetoFront` is used to indicate the name of the optional reference front:
```jaava
    Problem<DoubleSolution> problem;
    Algorithm<List<DoubleSolution>> algorithm;
    CrossoverOperator<DoubleSolution> crossover;
    MutationOperator<DoubleSolution> mutation;
    SelectionOperator<List<DoubleSolution>, DoubleSolution> selection;
    String referenceParetoFront = "" ;
```
The next group of sentences are for processing the program arguments. A benchmark problem is solved by default:
``` java
    String problemName ;
    if (args.length == 1) {
      problemName = args[0];
    } else if (args.length == 2) {
      problemName = args[0] ;
      referenceParetoFront = args[1] ;
    } else {
      problemName = "org.uma.jmetal.problem.multiobjective.zdt.ZDT1";
      referenceParetoFront = "jmetal-problem/src/test/resources/pareto_fronts/ZDT1.pf" ;
    }
```
Next the problem is loaded from its class name:
```java
    problem = ProblemUtils.<DoubleSolution> loadProblem(problemName);
```
Then the parameters and the algorithm are configured:
```java 
    double crossoverProbability = 0.9 ;
    double crossoverDistributionIndex = 20.0 ;
    crossover = new SBXCrossover(crossoverProbability, crossoverDistributionIndex) ;

    double mutationProbability = 1.0 / problem.getNumberOfVariables() ;
    double mutationDistributionIndex = 20.0 ;
    mutation = new PolynomialMutation(mutationProbability, mutationDistributionIndex) ;

    selection = new BinaryTournamentSelection<DoubleSolution>(new RankingAndCrowdingDistanceComparator<DoubleSolution>());

    algorithm = new NSGAIIBuilder<DoubleSolution>(problem, crossover, mutation)
        .setSelectionOperator(selection)
        .setMaxIterations(250)
        .setPopulationSize(100)
        .build() ;
```
The last step is to run the algorithm and to write the obtained solutions into two files and optionally to print the values of all the available quality indicators:
```java
    AlgorithmRunner algorithmRunner = new AlgorithmRunner.Executor(algorithm)
        .execute() ;

    List<DoubleSolution> population = algorithm.getResult() ;
    long computingTime = algorithmRunner.getComputingTime() ;

    JMetalLogger.logger.info("Total execution time: " + computingTime + "ms");

    printFinalSolutionSet(population);
    if (!referenceParetoFront.equals("")) {
      printQualityIndicators(population, referenceParetoFront) ;
    }
  }
```

### Running an algorithm from an IDE
Once you have configured your algorithm you can use your favorite IDE to execute them. For example, in the case of IntellJ Idea you can select the runner class name and select the option "Run 'NSGAIIRunner.main()'" clicking with the left mouse button if you intend to run NSGA-II:
![Running with IntellJ Idea](https://github.com/jMetal/jMetalDocumentation/blob/master/figures/runningNSGAIIRunnerInIntelliJIdea.png)

As as result of the execution, the following messages are printed into the output console:
```
jul 27, 2015 4:21:59 PM org.uma.jmetal.runner.multiobjective.NSGAIIRunner main
INFORMACIÓN: Total execution time: 1147ms
jul 27, 2015 4:21:59 PM org.uma.jmetal.runner.AbstractAlgorithmRunner printFinalSolutionSet
INFORMACIÓN: Random seed: 1438006918503
jul 27, 2015 4:21:59 PM org.uma.jmetal.runner.AbstractAlgorithmRunner printFinalSolutionSet
INFORMACIÓN: Objectives values have been written to file FUN.tsv
jul 27, 2015 4:21:59 PM org.uma.jmetal.runner.AbstractAlgorithmRunner printFinalSolutionSet
INFORMACIÓN: Variables values have been written to file VAR.tsv
jul 27, 2015 4:22:00 PM org.uma.jmetal.runner.AbstractAlgorithmRunner printQualityIndicators
INFORMACIÓN: 
Hypervolume (N) : 0.6594334269577787
Hypervolume     : 0.6594334269577787
Epsilon (N)     : 0.012122558511198256
Epsilon         : 0.012122558511198256
GD (N)          : 2.054388435747992E-4
GD              : 2.054388435747992E-4
IGD (N)         : 1.8304524180524584E-4
IGD             : 1.8304524180524584E-4
IGD+ (N)        : 0.003808931172199927
IGD+            : 0.003808931172199927
Spread (N)      : 0.34070732976112383
Spread          : 0.34070732976112383
R2 (N)          : 0.13179198315493879
R2              : 0.13179198315493879
Error ratio     : 1.0
```

The results tagged with `(N)` indicates that the fronts are normalized before being computed.


### Running an algorithm from the command line
If you plan to run a jMetal algorithm from the command line you have to take into account the following requirements:

1. Build the project with `mvn package`. This will create, for each subproject (i.e, `jmetal-core`, `jmetal-problem`, `jmetal-algorithm`, and `jmetal-exec`), a jar file with all the dependences.
2. Add the needed jar files to the class. For example, if you are want to test an algorithm on a benchmark problem all the jars will required. You have at least two ways of doing it. One is to set the `CLASSPATH` environment variable:

```
export CLASSPATH=jmetal-core/target/jmetal-core-5.0-jar-with-dependencies.jar:jmetal-problem/target/jmetal-problem-5.0-jar-with-dependencies.jar:jmetal-exec/target/jmetal-exec-5.0-jar-with-dependencies.jar:jmetal-problem/target/jmetal-problem-5.0-jar-with-dependencies.jar
```
  
  Then you can execute an algorithm this way (we are going to execute NSGA-II):
  
```
java org.uma.jmetal.runner.multiobjective.NSGAIIRunner 
```
3. The other alternative is to add the jars to the `java` command:
  
 ```
java -cp jmetal-exec/target/jmetal-exec-5.0-SNAPSHOT-jar-with-dependencies.jar:jmetal-core/target/jmetal-core-5.0-SNAPSHOT-jar-with-dependencies.jar:jmetal-problem/target/jmetal-problem-5.0-SNAPSHOT-jar-with-dependencies.jar:jmetal-algorithm/target/jmetal-algorithm-5.0-Beta-35-jar-with-dependencies.jar org.uma.jmetal.runner.multiobjective.NSGAIIRunner
 ```
 
 This example execute NSGA-II with the default parameters. If you want to solve a given problem its class name must be provided as an argument. For example, to solve the benchmark problem `ZDT4` the command would be:
 
 ```
 java org.uma.jmetal.runner.multiobjective.NSGAIIRunner org.uma.jmetal.problem.multiobjective.zdt.ZDT4
 ```
 
and the output will be similar to this:
```
jul 27, 2015 6:48:27 PM org.uma.jmetal.runner.multiobjective.NSGAIIRunner main
INFORMACIÓN: Total execution time: 683ms
jul 27, 2015 6:48:27 PM org.uma.jmetal.runner.AbstractAlgorithmRunner printFinalSolutionSet
INFORMACIÓN: Random seed: 1438015706581
jul 27, 2015 6:48:27 PM org.uma.jmetal.runner.AbstractAlgorithmRunner printFinalSolutionSet
INFORMACIÓN: Objectives values have been written to file FUN.tsv
jul 27, 2015 6:48:27 PM org.uma.jmetal.runner.AbstractAlgorithmRunner printFinalSolutionSet
INFORMACIÓN: Variables values have been written to file VAR.tsv
```
 
In the case of problems having a known Pareto front (or a Pareto front approximation), adding the file containing it allows to apply the available quality indicators to the obtained front. This way, the command to solve ZDT4 would be:
```
java org.uma.jmetal.runner.multiobjective.NSGAIIRunner org.uma.jmetal.problem.multiobjective.zdt.ZDT4 jmetal-problem/src/test/resources/pareto_fronts/ZDT4.pf
```
and this would be output:
```
jul 27, 2015 6:49:21 PM org.uma.jmetal.runner.multiobjective.NSGAIIRunner main
INFORMACIÓN: Total execution time: 598ms
jul 27, 2015 6:49:21 PM org.uma.jmetal.runner.AbstractAlgorithmRunner printFinalSolutionSet
INFORMACIÓN: Random seed: 1438015760471
jul 27, 2015 6:49:21 PM org.uma.jmetal.runner.AbstractAlgorithmRunner printFinalSolutionSet
INFORMACIÓN: Objectives values have been written to file FUN.tsv
jul 27, 2015 6:49:21 PM org.uma.jmetal.runner.AbstractAlgorithmRunner printFinalSolutionSet
INFORMACIÓN: Variables values have been written to file VAR.tsv
jul 27, 2015 6:49:21 PM org.uma.jmetal.runner.AbstractAlgorithmRunner printQualityIndicators
INFORMACIÓN: 
Hypervolume (N) : 0.6584874391103687
Hypervolume     : 0.658491021119803
Epsilon (N)     : 0.014508161683056214
Epsilon         : 0.014508161681605389
GD (N)          : 1.7281971372005978E-4
GD              : 1.7281858245371445E-4
IGD (N)         : 1.9833943989483466E-4
IGD             : 1.9833851420211548E-4
IGD+ (N)        : 0.00425088535021156
IGD+            : 0.004250860866635309
Spread (N)      : 0.4449171015114183
Spread          : 0.44491700055639544
R2 (N)          : 0.13208551920620412
R2              : 0.13208472309027727
Error ratio     : 1.0
```
