Car Service:
Wednesday 25th




---


### 1. Reduce repetition across test setup

A lot of each test is doing the same three things:

- create hospital company
    
- create viewer
    
- create one visible record
    
- create one out-of-scope record
    

That repetition makes the file longer than it needs to be and makes intent slightly harder to scan.

I’d pull that into a small helper that returns a standard fixture set, for example:

```php
function createScopedApplicationFixtures(array $visibleAttributes = []): array
{
    $company = createHospitalCompany();
    $viewer = createApplicationPagesUser($company);

    $visibleApplication = createApplicationPagesRecord($company, $visibleAttributes);
    $outsideApplication = createApplicationPagesRecord(createHospitalCompany());

    return compact('company', 'viewer', 'visibleApplication', 'outsideApplication');
}
```

That lets the tests focus on what differs.

---

### 2. The test names are a bit too implementation-heavy

They are accurate, but a little verbose and sometimes mix multiple concerns.

For example:

- `renders the applications list with scoped records and request filters`
    
- `renders the new application route with a null application and new route flag`
    

These are fine, but could be a bit more behaviour-focused:

- `shows only scoped applications on the index page`
    
- `marks the create page as a new application route`
    
- `omits transfer hospitals for users without transfer permission`
    
- `includes eligible transfer hospitals for authorised users`
    

That tends to make failures easier to interpret quickly.

---

### 3. The assertions on request filters are useful, but repetitive

You assert `request.name`, `request.sort`, and `request.direction` repeatedly. That is not wrong, but it is noisy.

You could wrap that in a custom assertion helper:

```php
function assertRequestFilters(Assert $page, array $filters): Assert
{
    foreach ($filters as $key => $value) {
        $page->where("request.{$key}", $value);
    }

    return $page;
}
```

Then inside the test:

```php
->assertInertia(fn (Assert $page) => assertRequestFilters($page
    ->component('applications/index')
    ->where('application', null)
    ->where('is_new_route', false)
    ->has('applications.data', 1)
    ->where('applications.data.0.id', $visibleApplication->id), [
        'name' => 'Alice',
        'sort' => 'created_at',
        'direction' => 'asc',
    ])
    ->has('reference_data.type_of_care')
    ->missing('transfer_hospitals')
    ->etc()
)
```

Not essential, but it improves readability if this pattern appears in other test files too.

---

### 5. The `transfer_hospitals` assertion is harder to read than it needs to be

This part works, but it is dense:

```php
->where('transfer_hospitals', fn ($hospitals) => collect($hospitals)->contains(
    fn (array $hospital) => $hospital['id'] === $otherHospital->id
        && $hospital['name'] === $otherHospital->name
) && !collect($hospitals)->contains(fn (array $hospital) => $hospital['id'] === $currentHospital->id)
    && !collect($hospitals)->contains(fn (array $hospital) => $hospital['id'] === $providerCompany->id))
```

I would rewrite that for clarity:

```php
->where('transfer_hospitals', function ($hospitals) use ($otherHospital, $currentHospital, $providerCompany) {
    $ids = collect($hospitals)->pluck('id');

    return $ids->contains($otherHospital->id)
        && ! $ids->contains($currentHospital->id)
        && ! $ids->contains($providerCompany->id);
})
```

That is much easier to read when revisiting the test later.

You could also assert the exact returned IDs if the list is intentionally small and deterministic.

---

### 6. Consider asserting more about the view route payload

In the view route tests, you assert:

- selected application id
    
- applications list contains one item
    

That is good, but if the page is expected to include the selected application in a specific resource shape, you could assert a couple more meaningful fields on `application.data`, not just the ID.

For example:

```php
->where('application.data.id', $visibleApplication->id)
->where('application.data.first_name', 'Ava')
->where('application.data.last_name', 'Patient')
```

That depends on how much value the page serializer adds. If it is just a pass-through, maybe not needed. If it transforms fields, it is worth asserting.


---

### 8. Use factories for `Application` if possible

This helper:

```php
return Application::query()->create([
    ...
]);
```

works, but if you have an `ApplicationFactory`, it would usually be cleaner and more expressive:

```php
return Application::factory()
    ->for($company)
    ->for($creator, 'creator')
    ->create([
        ...
        ...$attributes,
    ]);
```

That keeps test data closer to the model’s intended creation path and makes future changes easier.

If the factory does not exist yet, this file is not necessarily the place to build a huge one, but I would lean that way.

---

### 9. Consider grouping these as page access tests

The file currently mixes a few concerns:

- list route
    
- new route
    
- view route
    
- permission-driven payload differences
    
- 404 scope enforcement
    

That is acceptable, but if the file grows further, I’d split it into sections or separate files:

- `ApplicationIndexPageTest`
    
- `ApplicationViewPageTest`
    

Or at least add comments grouping behaviours.

---





Job to Apply for:
https://www.linkedin.com/jobs/view/4377110300/?trackingId=pUHbpsDyXQWpPSdEqlmo2g%3D%3D&refId=hw%2Bv1%2B55F4kyOiRIuhhUVA%3D%3D&midToken=AQEzzsZtjvla8Q&midSig=1h1zF_9OLbNs81&trk=eml-email_job_alert_digest_01-primary_job_list-0-job_posting_0_jobid_4377110300_ssid_15402721724_fmid_35t8t3~mma2qvgq~2g&trkEmail=eml-email_job_alert_digest_01-primary_job_list-0-job_posting_0_jobid_4377110300_ssid_15402721724_fmid_35t8t3~mma2qvgq~2g-null-35t8t3~mma2qvgq~2g-null-null&eid=35t8t3-mma2qvgq-2g&otpToken=MTMwYzFmZTExNDJmYzljMGIzMjQwNGVkNDExY2UyYjU4ZmNmZDE0Mjk4YWE4ODYxNzdjNzA4NmQ0ZTVhNThmMmY2ZGY4NmJkNTVjN2M0ODY2NzlmN2NmODlhMTc1NDMyZGViMWJmOTk1OGNjZGEyNTg2ZmIsMSwx


# Home Loan Application

| Item               | 2024     | 2025      |
| ------------------ | -------- | --------- |
| Taxable Income     | 6,534.00 | 10,216.00 |
| Depreciation       | 1,157.65 | 0         |
| Interest Add Backs | 0        | 0         |
| Operating Expenses | 5,269.09 | 1721<br>  |








Keyboards I like:
- [Charybdis MK2](https://bastardkb.com/product/charybdis-mk2-prebuilt-preorder/)
- [Ergokeyboards Crosses/Bridges Keyboard](https://ergokeyboards.com/products/crosses-modular-keyboard?variant=50272542228762)
- ZSA Moonlander



Good switch upgrades:
- Gateron Baby Kangaroo Switch
- K Pro Banana Switch