```sql
## Facilities
select facilities.id,
       facilities.company_id,
       companies.name as company_name,
       facilities.name,
       facilities.street_number,
       facilities.street,
       facilities.city,
       facilities.state,
       facilities.country,
       facilities.postcode,
       facilities.notification_emails,
       facilities.phone_number,
       facilities.website,
       facilities.fax_number,
       facilities.geohash,
       facilities.latitude,
       facilities.longitude,
       facilities.govt_subsidised,
       facilities.cbc_engaged_provider,
       facilities.benevolent_provider,
       facilities.carers_gateway,
       facilities.dementia_care,
       facilities.removed,
       facilities.refreshed_at,
       facilities.created_at,
       facilities.updated_at
from facilities
         inner join companies on facilities.company_id = companies.id;


## facilities.accommodation_types
select facilities_relationships.facility_id,
       accommodation_types.name,
       facilities_relationships.created_at,
       facilities_relationships.updated_at
from facilities_relationships
         inner join accommodation_types on facilities_relationships.type_id = accommodation_types.id
where type_name = 'accommodation_types';

# facilities.specific_service_deliveries
select facilities_relationships.facility_id,
       specific_service_deliveries.name,
       facilities_relationships.created_at,
       facilities_relationships.updated_at
from facilities_relationships
         inner join specific_service_deliveries on facilities_relationships.type_id = specific_service_deliveries.id
where type_name = 'specific_service_deliveries';


# facilities.roon_types
select facilities_relationships.facility_id,
       facilities_relationships.type_name,
       room_types.name,
       facilities_relationships.created_at,
       facilities_relationships.updated_at
from facilities_relationships
         inner join room_types on facilities_relationships.type_id = room_types.id
where type_name = 'room_types';


# vacancies
select
    vacancies.id,
    vacancies.name,
    vacancies.carers_gateway,
    vacancies.company_id,
    vacancies.facility_id,
    vacancies.cbc_engaged_provider,
    vacancies.dementia_care,
    vacancies.gender,
    vacancies.geohash,
    vacancies.latitude,
    vacancies.longitude,
    vacancies.govt_subsidised,
    vacancies.created_at,
    vacancies.updated_at
from vacancies;

# vacancies.accommodation_types
select vacancies_relationships.vacancy_id,
       accommodation_types.name,
       vacancies_relationships.created_at,
       vacancies_relationships.updated_at
from vacancies_relationships
         inner join accommodation_types on vacancies_relationships.type_id = accommodation_types.id
where vacancies_relationships.type_name = 'accommodation_types';

# vacancies.room_types
select vacancies_relationships.vacancy_id,
       room_types.name,
       vacancies_relationships.created_at,
       vacancies_relationships.updated_at
from vacancies_relationships
         inner join room_types on vacancies_relationships.type_id = room_types.id
where vacancies_relationships.type_name = 'room_types';

# vacancies.specific_service_deliveries
select vacancies_relationships.vacancy_id,
       specific_service_deliveries.name,
       vacancies_relationships.created_at,
       vacancies_relationships.updated_at
from vacancies_relationships
         inner join specific_service_deliveries on vacancies_relationships.type_id = specific_service_deliveries.id
where vacancies_relationships.type_name = 'specific_service_deliveries';

```