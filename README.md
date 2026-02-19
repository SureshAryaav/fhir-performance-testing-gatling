# fhir-performance-testing-gatling
fhir-performance-testing-gatling
public class EHRWorkloadSimulation extends Simulation {

    // Simulate realistic EHR usage pattern
    // Morning shift: high patient admissions
    // Afternoon: mix of queries and updates  
    // Night: low activity

    ScenarioBuilder patientAdmission = scenario("Patient Admission Workflow")
        .exec(http("Create FHIR Patient").post("/Patient")...)
        .pause(2)
        .exec(http("Create Encounter").post("/Encounter")...)
        .pause(1)
        .exec(http("Add Observation").post("/Observation")...);

    ScenarioBuilder patientQuery = scenario("Clinical Data Query")
        .exec(http("Get Patient").get("/Patient/#{id}")...)
        .exec(http("Get Patient History")
            .get("/Observation?patient=#{id}&_sort=-date&_count=20")...);

    {
        setUp(
            // Morning spike simulation
            patientAdmission.injectOpen(
                rampUsers(50).during(Duration.ofMinutes(5))
            ),
            patientQuery.injectOpen(
                constantUsersPerSec(10).during(Duration.ofMinutes(10))
            )
        ).assertions(
            global().responseTime().percentile(95).lt(2000),
            global().failedRequests().percent().lt(1.0)
        );
    }
}
```

**Add a `results/` folder with a sample HTML report screenshot.** Visual proof you've run it.

