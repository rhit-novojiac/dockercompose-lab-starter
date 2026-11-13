# Spike Worksheet

Name: Alexei Novojilov

Fill this in as you work through the stages. Every **Prediction** gets written
down *before* you run anything.

You are not graded on whether a prediction was right. You are graded on having
made one and being able to explain the gap.

This file is your Spike deliverable. Commit it on your branch and push it
when you are done.

---

## Stage 1 — Why did it fail?

**What happened** (paste the lines from the api where it gives up):

> startup check FAILED: cannot reach the database -- connection to server at "db" (172.22.0.2), port 5432 failed: Connection refused

**Q1.** `docker compose ps` reports the `db` container as running, and the api
still could not reach it. What is the difference between those two statements?

> The db is running, meaning that its working, but its not able to be connected to by the api via the port its supposed to. It isn't ready to be connected to yet

**Q2.** Look back at the map you drew in the Lab. Did you predict this failure?
If you predicted a different symptom, what did you expect instead?

> I did not expect this issue popping up in this way. I expected that since the migration hasn't run it would either fail to connect because the migration is doing things with the db or it would connect but not be able to do anything.

---

## Stage 2 — Make the api wait for readiness

**Q3.** You added a healthcheck and changed how `depends_on` works. In your own
words, what is `pg_isready` actually testing, and why can Compose not work that
out by itself?

> pg_isready is checking to make sure that the service is able to be accessed by others. Compose cannot check this because it just checks if it is running or not. It is a simple container orchestrator.

**Q4.** A classmate says "just put a retry loop in the api and you don't need
any of this." Give one argument for their approach and one against it.

> That is more resource intensive than this, though this can be done easily.

---

## Stage 3 — Add the migration job

**Q5.** `flyway` needs a different `depends_on` condition than `db` does. Why?
What is different about a job that is supposed to finish?

> The DB is meant to be constantly running while flyway is meant to be a one and done kinda thing (one-shot batch job).

**Q6.** Run `docker compose ps -a` once the stack is up. What state is the
flyway container in? Is that a problem?

> It is in Exited (0) state. This is not a problem because we wanted it to run and then exit successfully (code 0).

---

## Stage 4 — Let the services talk

**PREDICTION 1.** `ML_SERVICE_URL` is `http://localhost:8080`, and the `ml`
service publishes port 8080. From inside the `api` container, will that URL
reach the ml service?

> Yes, I think that these will be able to talk to each other since they will be able tot alk via localhost

**What actually happened:**

> I forgot that localhost is per container. Since they couldn't access another containers localhost they were unable to communicate with each other. {"detail":"ML service unavailable: [Errno 111] Connection refused"}

**Q7.** Explain what `localhost` means from inside a container, and why
`http://ml:8080` works when `http://localhost:8080` does not.

> I forgot that localhost is per container. Since they couldn't access another containers localhost they were unable to communicate with each other

**Q8.** You can reach the ml service from your own browser at
`localhost:8080` because it publishes that port. Does the `api` need that
published port in order to call it? Remove the `ports:` block from `ml` and
find out.

> No, the "ports:" section just publishes that port to my machine.

---

## Stage 5 — Where the data lives

Add a player through the api first, so there is something to lose.

**PREDICTION 2.** With no `volumes:` on `db`, will your player still be there
after `docker compose down` and `docker compose up`?

> No, there is no persistant storage without volumes

**PREDICTION 3.** With a named volume, will it survive `docker compose down`?

> I believe it will

**PREDICTION 4.** With a named volume, will it survive `docker compose down -v`?

> I believe it won't

**What actually happened, for all three:**

> I was correct with my predictions

**Q9.** `api` has `./app:/app` and `db` has
`postgres-data:/var/lib/postgresql/data`. Both sit under `volumes:`. What is
the difference between them, and why does each service want the kind it has?

> One is a physical folder on my host system while the other is a storage system used by docker. API wants the physical one for faster running of the code while the db wants the docker controlled one because it is easier to manage postgres that way.

---

## Timing

How long did a full `docker compose down -v` followed by `docker compose up`
take, from nothing to the api answering a request?

> It was very fast. I believe that I got them both to run successfully in under 30 seconds. Which is much much better than the current system. 

How does that compare against the success measure you set for yourself in the
Lab?

> The success measure I set for myself was 1-2 commands and only one terminal window. If we consider that a new employee would only have to run docker compose up, that means they only need one terminal window and one command. Which is very good.

---

## Analyze

**Q10.** Look at everything you changed. How much of it was application code,
and how much was configuration?

> Everything that I changed was configuration

**Q11.** Did your approach introduce any new problems? Anything that is now
slower, more complicated, or easier to get wrong?

> We may potentially lose data if we run docker compose down -v. Also, now logs are harder to find if something goes wrong.

**Q12.** What does someone have to know about this compose file that they would
not have had to know about the twelve-step README?

> They have to know about hostnames and how Compose automatically reads .env files.

---

## Reflect

**R1.** Did your implementation hit the success measure you set for yourself in
the Lab? If you missed it, by how much, and what would close the gap?

> It did reach it.

**R2.** Go back to the failures you predicted in step 3 of the Lab. Which ones
happened the way you described? Which showed up as a completely different
symptom than you expected?

> I believe that most of them worked the way I thought. I had expected a lot of failures regarding edge cases when certain parts try to call other parts that don't work. While I was right about the connection error with API calling ml before its ready, the rest I didn't see those errors for.

**R3.** Which of your predictions during the Spike were wrong? Pick the one that
surprised you most and explain what you had misunderstood.

> My prediction about connection via localhost was wrong. I had misunderstood that localhost is for the containers and not for my machine so while I am used to programs being able to talk via localhost, that isn't the case here.

**R4.** In the Lab you listed what a configuration file would need to be able to
express. Compare that against what you actually wrote. What did you miss, and
what turned out to be unnecessary?

> I was correct in everything I said. We do need the 4 systems to be able to talk with each other and a startup order. We did also use healthcheck and a config file. I missed the need for volumes.

**Optional.** Name one thing about this stack that would still stop you from
running it anywhere other than your own laptop.

>
