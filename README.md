# Clinical Trial Cohort Definition and Evaluation with CQL and CDS-Hooks

This repository contains a postman collection and simplified walkthrough of an automated clinical trial patient selection architecture as described [here](https://pubmed.ncbi.nlm.nih.gov/38682521/). The following diagram [(originally published here)](https://pubmed.ncbi.nlm.nih.gov/38682521/) gives an overview of the proposed architecture:
![Resource Overview Image Reviewed](https://github.com/1anja1/ClinicalTrialCQL/assets/109726549/baddf899-0a7c-48aa-b040-383ad52819ad)




The prerequisite for the walkthrough is for researchers to specify the criteria for the trial. In this example, an existing trial is used so the criteria are already defined. The criteria for [the chosen trial](https://www.clinicaltrials.gov/study/NCT04753502) are summarized here:
| Min Age | Max Age | Gender | Pregnancy Status | Tobacco Use | Inclusion Criteria | Exclusion Criteria|
|--|--|--|--|--|--|--|
| 18 | N/A  | female | true  | N/A | Diagnosed Appendicitis  |N/A  |


The criteria can, if available be entered via a UI. In this walkthrough the CQL generation step is omitted. The resulting CQL Library, Measure and Plandefinition are provided directly in the postman collection.


The following walkthrough uses the Postman collection [here](https://github.com/1anja1/ClinicalTrialCQL/blob/main/ClinicalTrialsCQL.postman_collection.json).

The necessary tools for the following steps are:
 - Postman (or similar)
 - Docker for local instance of CQF Ruler (alternatively: public [HAPI FHIR Server](http://hapi.fhir.org/baseR4))


## Steps
 1. Set up \FHIR Server \& CQF Ruler
 To set up a local instance of the CQF Ruler with Docker (1.) enter these commands into the terminal:

    ``` 
    docker pull alphora/cqf-ruler
    docker run -p 8080:8080 alphora/cqf-ruler
    ```
	or follow the guide [here](https://github.com/cqframework/cqf-ruler/wiki/Deployment)
 3. GET metadata to check server requirements
 4. POST test resources to the FHIR server
 5. POST ClinicalTrials Base Library, FHIRHelpers Library, the example trial Library, and Plandefinition  (In real system: Identify criteria and enter via UI)
 6. Check cds-hooks endpoint for active cds-services

 **Result:**
 ```
 {
    "services": [
        {
            "hook": "patient-view",
            "id": "PregnancyAppendicitisClinicalTrialPlanDefinition",
            "prefetch": {
                "item1": "Patient?_id={{context.patientId}}"
            }
        }
    ]
}
 ```
 7. Trigger cds-hook for different patients
    
	 **Result for patient who does not qualify (Patient 1&3)**
	 ```
	 {
	    "cards": []
    }
	 ```
	 **Result for patient who qualifies (Patient 2)**
	 ```
	 {
	    "cards": [
	        {
	            "summary": "Clinical Trial Qualification",
	            "detail": "The Patient qualifies for trial: Laparoscopic Treatment for Appendicitis During Pregnancy",
	            "indicator": "info",
	            "links": [
	                {
	                    "label": "Clinical Trial Details",
	                    "url": "https://clinicaltrials.gov/study/NCT04753502",
	                    "type": "absolute"
	                }
	            ]
	        }
	    ]
    } 

 9. PUT Measure
 10. GET \$evaluate-measure operation: The parameter reportType is set to subject-list to get a list of ids of the eligible patients
     
 **Result:**
 ```
 {
    "resourceType": "MeasureReport",
    "contained": [
        {
            "resourceType": "List",
            "id": "30c34a37-7043-445a-87f9-3aa58971818b",
            "entry": [
                {
                    "item": {
                        "reference": "clinicalTrials-patient-2"
                    }
                }
            ]
        }
    ],
    "status": "complete",
    "type": "subject-list",
    "measure": "http://cql/Measure/PregnancyAppendicitisClinicalTrialMeasure",
    "date": "2024-03-20T07:15:01+00:00",
    "period": {
        "start": "1923-01-01T00:00:00+00:00",
        "end": "2023-09-16T00:00:00+00:00"
    },
    "group": [
        {
            "population": [
                {
                    "code": {
                        "coding": [
                            {
                                "code": "initial-population"
                            }
                        ]
                    },
                    "count": 1,
                    "subjectResults": {
                        "reference": "#30c34a37-7043-445a-87f9-3aa58971818b"
                    }
                }
            ]
        }
    ]
}
```





