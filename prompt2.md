# Prompt #02 — Station & Pricing Pydantic Schemas

## 📌 Milestone 1: Domain Modeling & Schema

This document contains the complete **Pydantic V2** data validation models for Gas Stations in the **GasTrack** application.

---

## 🛠️ Python Implementation (`app/schemas/gas_station.py`)

```python
from datetime import datetime
from typing import List, Optional
from pydantic import BaseModel, ConfigDict, Field, field_validator

# Standard supported cylinder sizes in kg (3kg, 6kg, 12.5kg, 25kg, 50kg)
ALLOWED_CYLINDER_SIZES = {3.0, 6.0, 12.5, 25.0, 50.0}


class StationBase(BaseModel):
    """Base schema for Gas Station with common attributes."""

    name: str = Field(
        ...,
        min_length=2,
        max_length=100,
        description="Name of the gas refill station",
        examples=["Lekki Central Gas Hub"],
    )
    address: str = Field(
        ...,
        min_length=5,
        max_length=255,
        description="Physical street address of the station",
        examples=["12 Admiralty Way, Lekki Phase 1, Lagos"],
    )
    contact_phone: str = Field(
        ...,
        min_length=7,
        max_length=20,
        description="Contact telephone number",
        examples=["+2348012345678"],
    )
    price_per_kg: float = Field(
        ...,
        gt=0,
        description="Price per kilogram of cooking gas in local currency (must be > 0)",
        examples=[1200.00],
    )
    supported_cylinder_sizes: List[float] = Field(
        ...,
        min_length=1,
        description="List of cylinder sizes in kg supported by the station",
        examples=[[6.0, 12.5, 25.0, 50.0]],
    )
    is_active: bool = Field(
        default=True,
        description="Status flag indicating if the gas station is currently operating",
    )

    @field_validator("supported_cylinder_sizes")
    @classmethod
    def validate_cylinder_sizes(cls, sizes: List[float]) -> List[float]:
        """Custom validator to ensure provided cylinder sizes are valid standard sizes."""
        invalid_sizes = [size for size in sizes if size not in ALLOWED_CYLINDER_SIZES]
        if invalid_sizes:
            valid_list = sorted(list(ALLOWED_CYLINDER_SIZES))
            raise ValueError(
                f"Invalid cylinder size(s): {invalid_sizes}. "
                f"Allowed standard sizes are: {valid_list}"
            )
        # Deduplicate and sort sizes
        return sorted(list(set(sizes)))


class StationCreate(StationBase):
    """Schema for creating a new gas station (inherits all validation rules from StationBase)."""

    pass


class StationUpdate(BaseModel):
    """Schema for updating an existing gas station (all fields are optional)."""

    name: Optional[str] = Field(None, min_length=2, max_length=100)
    address: Optional[str] = Field(None, min_length=5, max_length=255)
    contact_phone: Optional[str] = Field(None, min_length=7, max_length=20)
    price_per_kg: Optional[float] = Field(None, gt=0)
    supported_cylinder_sizes: Optional[List[float]] = Field(None, min_length=1)
    is_active: Optional[bool] = None

    @field_validator("supported_cylinder_sizes")
    @classmethod
    def validate_cylinder_sizes(
        cls, sizes: Optional[List[float]]
    ) -> Optional[List[float]]:
        if sizes is None:
            return sizes
        invalid_sizes = [size for size in sizes if size not in ALLOWED_CYLINDER_SIZES]
        if invalid_sizes:
            valid_list = sorted(list(ALLOWED_CYLINDER_SIZES))
            raise ValueError(
                f"Invalid cylinder size(s): {invalid_sizes}. "
                f"Allowed standard sizes are: {valid_list}"
            )
        return sorted(list(set(sizes)))


class StationResponse(StationBase):
    """Schema for gas station responses sent back to API clients."""

    id: int = Field(..., description="Unique database primary key identifier")
    created_at: datetime = Field(
        ..., description="Timestamp when the station record was created"
    )

    # Configured for ORM compatibility in Pydantic v2
    model_config = ConfigDict(from_attributes=True)
```

---

## 🔍 Key Pydantic V2 Concepts Explained

### 1. `Field(gt=0)`
Enforces strict quantitative rules. If a user submits `price_per_kg = -500` or `price_per_kg = 0`, Pydantic immediately returns an HTTP 422 Unprocessable Entity error with a clear message: `"Input should be greater than 0"`.

### 2. `@field_validator("supported_cylinder_sizes")`
A custom validator decorator in Pydantic V2. It intercepts the incoming array of floats and checks if every element exists in our `ALLOWED_CYLINDER_SIZES` set `{3.0, 6.0, 12.5, 25.0, 50.0}`. It also automatically removes duplicates and sorts the array.

### 3. `model_config = ConfigDict(from_attributes=True)`
In Pydantic V2, `ConfigDict(from_attributes=True)` replaces Pydantic V1's `class Config: orm_mode = True`. This allows FastAPI to automatically read attributes directly from SQLAlchemy database objects (`station.id`, `station.name`) when converting database records into JSON HTTP responses.
