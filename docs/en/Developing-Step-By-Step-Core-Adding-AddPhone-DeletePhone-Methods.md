# AddPhone and DeletePhone Methods

We are adding two more methods to IPersonAppService interface as shown
below:

```csharp
Task DeletePhone(EntityDto<long> input);
Task<PhoneInPersonListDto> AddPhone(AddPhoneInput input);
```

We could create a new, separated IPhoneAppService. It's your choice.
But, we can consider Person as an aggregate and add phone related
methods here. AddPhoneInput DTO is shown below:

```csharp
public class AddPhoneInput
{
    [Range(1, int.MaxValue)]
    public int PersonId { get; set; }

    [Required]
    public PhoneType Type { get; set; }

    [Required]
    [MaxLength(PhoneConsts.MaxNumberLength)]
    public string Number { get; set; }
}
```

Now, we can implement these methods:

```csharp
public async Task DeletePhone(EntityDto<long> input)
{
    await _phoneRepository.DeleteAsync(input.Id);
}

public async Task<PhoneInPersonListDto> AddPhone(AddPhoneInput input)
{
    var person = _personRepository.Get(input.PersonId);
    await _personRepository.EnsureCollectionLoadedAsync(person, p => p.Phones);

    var phone = ObjectMapper.Map<Phone>(input);
    person.Phones.Add(phone);

    await CurrentUnitOfWork.SaveChangesAsync();

    return ObjectMapper.Map<PhoneInPersonListDto>(phone);
}
```

And create mapping in CustomDtoMapper.cs:

```csharp
configuration.CreateMap<AddPhoneInput, Phone>();
```

(Note: We injected **IRepository&lt;Phone, long&gt;** in the constructor
and set to \_phoneRepository field)

**DeletePhone** method is simple. It only deletes phone with given id.

**AddPhone** method **gets** the person from the database and adds a new phone
to the person's Phones collection. After that **save changes** is called. Saving changes
causes inserting newly added phone to the database and getting back its **Id**.
Because, we are returning a DTO that contains newly created phone's 
information (including the Id), it would be assigned before the mapping in the last line.
(Note however that;  it's not normally needed to call
CurrentUnitOfWork.SaveChangesAsync since it is automatically called at the end
of the method. We called it in the method bacause we need to save entity
and get back its Id immediately. See [UOW
document](https://aspnetboilerplate.com/Pages/Documents/Unit-Of-Work#DocAutoSaveChanges)
for more.)

There may be different approaches for AddPhone method. You can directly
work with a **phone repository** to insert new phone. They all have
different pros and cons. It's your choice.

## Next

* [Edit Mode For Phone Numbers](Developing-Step-By-Step-Core-Edit-Mode-For-Phone-Numbers)
