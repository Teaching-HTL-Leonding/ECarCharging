# E-Car Charging

![Hero Image](./hero.png)

This exercise will require you to create an *ASP.NET Core Minimal API* with *Entity Framework Core* for maintaining the accounting infrastructure for charging infrastructure for e-cars. The database will store data related to customers, cars, and chargings, and the API will expose endpoints for creating, updating, and querying this data.

## Basic Requirements

Your solution must include the following components:

1. A `Customer` entity with the following properties:
   * `Id`: A unique identifier for the customer, assigned by the system
   * `Company Name`: The company name of the customer (mandatory, maximum length 1024 characters)
   * `Address`: The address of the customer (mandatory, maximum length 1024 characters)
   * `IsActive`: A boolean indicating whether the customer is active (`true` by default). This value could turn to `false` if the customer e.g. doesn't pay bills.
   * `CurrentPricePerKwInCent`: The current price per kW for the customer in Euro Cent (mandatory; `int` data type is ok). This price might change over time.

2. A `Car` entity with the following properties:
   * `Id`: A unique identifier for the car, assigned by the system
   * `LicensePlate`: The license plate of the car (mandatory, maximum length 20 characters)
   * `Make`: The make of the car (mandatory, maximum length 128 characters)
   * `Model`: The model of the car (mandatory, maximum length 128 characters)
   * Relation to the customer that the car is assigned to (mandatory)

3. A `ChargingProcess` entity with the following properties:
   * `Id`: A unique identifier for the charging process, assigned by the system
   * `StartDateTime`: The date and time when the charging process started (mandatory; `DateTime` data type is ok)
   * `EndDateTime`: The date and time when the charging process ended (is null during charging process; `DateTime` data type is ok)
   * `TotalKw`: The total kW that was charged (is null during charging process; `int` data type is ok)
   * `PricePerKwInCent`: The price per kW in Euro (mandatory; `int` data type is ok). This price is copied from the customer at the start of the charging process.
   * `TotalPriceInCent`: The total price for the charging process (is null during charging process; `int` data type is ok). This value must be calculated when the charging process ends. Formula: `TotalKw * PricePerKw`.
   * Relation to the car that was charged (mandatory)

4. A `DbContext` class, `ChargingDbContext`, which includes `DbSet` properties for the entities mentioned above.

5. An API with the following endpoints:
   * POST `/customers`: An endpoint that accepts a JSON payload with the customer data. This endpoint should create a new `Customer` records and return at least its `Id` (you can return the entire customer record if you want).
   * POST `/customers/{id}/cars`: An endpoint that accepts a JSON payload with the car data. This endpoint should create a new `Car` and return at least its `Id` (you can return the entire car record if you want).
   * POST `/cars/{carId}/charging/start`: An endpoint that starts the charging process for a given car. This endpoint does not get any additional data except the car ID. It creates a new `ChargingProcess` record and sets `StartDateTime` and `PricePerKwInCent`.
   * PATCH `/cars/{carId}/charging/end`: An endpoint that ends the current charging process for a given car. The current charging process is the one with `EndDateTime == null`. This endpoint receives the `TotalKw` for the charging process. It updates the currently active charging process for the given car by setting `EndDateTime` to the current system time, `TotalKw`, and `TotalPriceInCent`.

All data must be stored in a SQL Server database using Entity Framework Core.

Here is a diagram of the database schema:

[![](https://mermaid.ink/img/pako:eNp9UsFugzAM_ZUo5_YHuFV0B9RtQmpPFRePuDQaJJVjhirg32foGANVu9nPL89-dlqde4M60kh7CwVBlTml4jqwr5BUO2RKWccqMY84MFlXqNhXN3D3d6hwge-MIQzhgX14XyI4lYRdzvYLZ7W4JkLHKdkcU6RDk7hY8oHQjxPAf81f5ZULmJbAy-5v8LkCxF05q16BCoFT8rkM-ayDEUm2FaojA_FespOdLP7WXpxZVgaFk2coD80MPDP3hzqW17anxXfddtu14xoi5RsXpqVMhZWTSOUDgsLTGy0KFVgjZx0dZpqvKKPqSEKDF6hLznTmeqFCzf54d7mOmGrc6Po2uPz5Czq6QBkElVOfvZ_y_htimL9w?type=png)](https://mermaid.live/edit#pako:eNp9UsFugzAM_ZUo5_YHuFV0B9RtQmpPFRePuDQaJJVjhirg32foGANVu9nPL89-dlqde4M60kh7CwVBlTml4jqwr5BUO2RKWccqMY84MFlXqNhXN3D3d6hwge-MIQzhgX14XyI4lYRdzvYLZ7W4JkLHKdkcU6RDk7hY8oHQjxPAf81f5ZULmJbAy-5v8LkCxF05q16BCoFT8rkM-ayDEUm2FaojA_FespOdLP7WXpxZVgaFk2coD80MPDP3hzqW17anxXfddtu14xoi5RsXpqVMhZWTSOUDgsLTGy0KFVgjZx0dZpqvKKPqSEKDF6hLznTmeqFCzf54d7mOmGrc6Po2uPz5Czq6QBkElVOfvZ_y_htimL9w)

## Advanced Requirements

![Charging](./charging.png)

1. Add a new endpoint POST `/customers/{id}/deactivate`: An endpoint that sets a customer's `IsActive` flag to `false`. The endpoint must return *Bad Request* if the customer is already inactive.

2. Add the following checks to the existing endpoint `/cars/{carId}/charging/start`:
   * A charging process can only start if the customer is active.
   * A charging process can only start if the car is not currently charging (i.e. no other charging record exists for the car with `EndDateTime == null`).

3. Add the following checks to the existing endpoint `/cars/{carId}/charging/end`:
   * A charging process can only end if there is an active charging process for the car (i.e. a charging record exists for the car with `EndDateTime == null`).
   * `TotalKw` must be greater than 0.

4. Add a new endpoint PATCH `/cars/{carId}/charging/cancel`: An endpoint that cancels the current charging process for a given car. The current charging process is the one with `EndDateTime == null`. It updates the currently active charging process for the given car by setting `EndDateTime` to the current system time, `TotalKw` to zero, and `TotalPriceInCent` to zero.
   * A charging process can only be cancelled if there is an active charging process for the car (i.e. a charging record exists for the car with `EndDateTime == null`).

5. Add a new endpoint GET `/customers/{id}?from=<date/time>&to=<date/time>`: An endpoint that returns the sum of `TotalKw` and `TotalPriceInCent` for all cars of the given customer. Only charging processes that started between the given `from` and `to` date/times should be considered. The endpoint must return *Bad Request* if `from` is greater than `to`.
