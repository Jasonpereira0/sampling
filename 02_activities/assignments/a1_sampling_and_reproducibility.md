# ASSIGNMENT: Sampling and Reproducibility in Python

Read the blog post [Contact tracing can give a biased sample of COVID-19 cases](https://andrewwhitby.com/2020/11/24/contact-tracing-biased/) by Andrew Whitby to understand the context and motivation behind the simulation model we will be examining.

Examine the code in `whitby_covid_tracing.py`. Identify all stages at which sampling is occurring in the model. Describe in words the sampling procedure, referencing the functions used, sample size, sampling frame, any underlying distributions involved, and how these relate to the procedure outlined in the blog post.

Run the Python script file called whitby_covid_tracing.py as is and compare the results to the graphs in the original blog post. Does this code appear to reproduce the graphs from the original blog post?

Modify the number of repetitions in the simulation to 1000 (from the original 50000). Run the script multiple times and observe the outputted graphs. Comment on the reproducibility of the results.

Alter the code so that it is reproducible. Describe the changes you made to the code and how they affected the reproducibility of the script file. The output does not need to match Whitby’s original blogpost/graphs, it just needs to produce the same output when run multiple times

# Author: Jason Pereira

```
The simulated model involves sampling at several stages:
1. Initial population sampling - Total of 1,000 individuals (sampling frame) attending two events: 200 people at two weddings with 100 people attending each wedding and 800 people attending 80 brunches with 10 people attending each brunch. The initial population is defined as what is described in the blog post. The underlying distribution is deterministic allocation.

events = ['wedding'] * 200 + ['brunch'] * 800
ppl = pd.DataFrame({
    'event': events,
    'infected': False,
    'traced': np.nan
})

2. Sampling people with COVID-19 - Simple random sampling is used for 100 random people that have a 10% probability (ATTACK_RATE = 0.10) of being infected. The underlying distribution is unifromly random across all the event attendees. This is aligned to the information in the blog post.

infected_indices = np.random.choice(ppl.index, size=int(len(ppl) * ATTACK_RATE), replace=False)
ppl.loc[infected_indices, 'infected'] = True

3. First Contact Tracing Sampling - From the group of infected people, a random set of people are selected with a 20% probability (TRACE_SUCCESS = 0.20) of being infected. This sampling simualates as per the blog post where only a small set of cases are successfully traced because of resource limitations. The underlying distribution follows Bernoulli trials to determine the infected person.

ppl.loc[ppl['infected'], 'traced'] = np.random.rand(sum(ppl['infected'])) < TRACE_SUCCESS

4. Second Contact Tracing Sampling - In the same event, if at least two infections are identified as traced (SECONDARY_TRACE_TRESHOLD = 2), all the people at that event are tested and traced if found infected. This will trigger the event as high risk having multiple infections and ramps up the tracing effort. This follows the methodology of second contact tracing as mentioned in the blog post but depends on meeting specific conditions for secondary tracing to occur frequently enough. The underlying distribution follows deterministic inclusion.

event_trace_counts = ppl[ppl['traced'] == True]['event'].value_counts()
events_traced = event_trace_counts[event_trace_counts >= SECONDARY_TRACE_THRESHOLD].index
ppl.loc[ppl['event'].isin(events_traced) & ppl['infected'], 'traced'] = True

5. Repeated Simulation - The whole simulation is repeated 50,000 times. As per the blog post, this introduces bias resulting in overestimating the number of infections from traceable events that are conisdered as potential risk of spreading infection.

results = [simulate_event(m) for m in range(50000)]
props_df = pd.DataFrame(results, columns=["Infections", "Traces"])

However, when the python code is run, it is observed the output graph indicated an overlap between Infections from wedding and Traced to weddings indicating little bias and high correlation contradicting the results in the blog post. This could be due to the following factors:

1. Weddings are more likely than brunches to meet the threshold for secondary tracing due to their larger size, which would typically amplify bias toward overrepresenting weddings as infection sources. However, if primary tracing (20% success rate) fails to identify enough cases initially or if secondary tracing conditions are not frequently met, this amplification effect diminishes.

2. Repeating simulations 50,000 times introduces averaging that smooths out variability and randomness caused by infection sampling and tracing decisions. This leads to stable proportions of cases where infections and traces align closely but may misrepresent real-world dynamics where variability is larger and more complex.

Reducing the number of simulations to 1,000 from 50,000 does not change the resulting output significantly even when run several times likely due the same reasons mentioned above like averaging smoothing out variability casued by randomnness in sampling infected people and tracing.

Reproducibility:

In order to make the output reprodicable and consistent no matter how many times simulations are run, adding a set random seed to the script will accomplish this with the following code without altering the methodology of the simulation 

np.random.seed(42)

```


## Criteria

|Criteria|Complete|Incomplete|
|--------|----|----|
|Altercation of the code|The code changes made, made it reproducible.|The code is still not reproducible.|
|Description of changes|The author explained the reasonings for the changes made well.|The author did not explain the reasonings for the changes made well.|

## Submission Information

🚨 **Please review our [Assignment Submission Guide](https://github.com/UofT-DSI/onboarding/blob/main/onboarding_documents/submissions.md)** 🚨 for detailed instructions on how to format, branch, and submit your work. Following these guidelines is crucial for your submissions to be evaluated correctly.

### Submission Parameters:
* Submission Due Date: `HH:MM AM/PM - DD/MM/YYYY`
* The branch name for your repo should be: `sampling-and-reproducibility`
* What to submit for this assignment:
    * This markdown file (sampling_and_reproducibility.md) should be populated.
    * The `whitby_covid_tracing.py` should be changed.
* What the pull request link should look like for this assignment: `https://github.com/<your_github_username>/sampling/pull/<pr_id>`
    * Open a private window in your browser. Copy and paste the link to your pull request into the address bar. Make sure you can see your pull request properly. This helps the technical facilitator and learning support staff review your submission easily.

Checklist:
- [X] Create a branch called `sampling-and-reproducibility`.
- [X] Ensure that the repository is public.
- [X] Review [the PR description guidelines](https://github.com/UofT-DSI/onboarding/blob/main/onboarding_documents/submissions.md#guidelines-for-pull-request-descriptions) and adhere to them.
- [X] Verify that the link is accessible in a private browser window.

If you encounter any difficulties or have questions, please don't hesitate to reach out to our team via our Slack at `#cohort-5-help`. Our Technical Facilitators and Learning Support staff are here to help you navigate any challenges.
